<template>
	<div id="dragndrop" class="drag-and-drop" :class="{dragging: dragging}" @dragover.prevent>
		<span class="tip">放手导入，文件夹重新计算，文件追加</span>
	</div>
</template>

<script>
const sortData = (list) =>
	list.filter(Boolean).sort((a, b) => {
		let aName = parseInt(a.name.toUpperCase().match(/\d{1,}/));
		let bName = parseInt(b.name.toUpperCase().match(/\d{1,}/));
		console.log(aName, ' - ', bName);
		if (aName < bName) {
			return -1;
		}
		if (aName > bName) {
			return 1;
		}
		return 0;
	});

export default {
	mounted() {
		let elms = new Set();
		window.addEventListener('dragenter', (e) => {
			if (e.dataTransfer.effectAllowed === 'move') return;
			if (elms.size === 0) this.onDragEnter();
			elms.add(e.target);
		});
		window.addEventListener('drop', (e) => {
			elms.delete(e.target);
			this.onFolderDrop(e);
		});
		window.addEventListener('dragleave', (e) => {
			elms.delete(e.target);
			if (elms.size === 0) this.onDragLeave();
		});
	},
	data: function () {
		return {
			dragging: false,
			nextList: [],
		};
	},
	methods: {
		onDragLeave() {
			this.dragging = false;
		},
		onDragEnter() {
			this.dragging = true;
		},
		async loadFileEntry(entry) {
			return new Promise((resolve) => {
				if (entry.isFile) {
					entry.file((file) => {
						if (['image/jpeg', 'image/png'].includes(file.type)) {
							resolve({
								name: file.name,
								file,
							});
						} else {
							return resolve(null);
						}
					});
				} else {
					return resolve(null);
				}
			});
		},
		async onFolderDrop(e) {
			e.preventDefault();

			this.dragging = false;
			if (e.dataTransfer.items) {
				// console.log(e.dataTransfer.items);
				let items = Array.from(e.dataTransfer.items);
				let nextList = [];
				// convert to entry -webkit-
				items = items.map((item) => item.webkitGetAsEntry());
				const fileOnly = items.every((item) => item.isFile);

				// 都是文件
				if (fileOnly) {
					nextList = await Promise.all(items.map((item) => this.loadFileEntry(item)));
					nextList = sortData(nextList);
					console.log(`再加 ${nextList.length} 张图片`);
					this.$message.success(`再加 ${nextList.length} 张图片`);
					this.$emit('add', {data: nextList});
					return;
				}

				const firstDir = items.find((item) => item.isDirectory);
				// 当前第一个文件夹
				if (firstDir) {
					let directoryReader = firstDir.createReader();
					// 读取子文件
					directoryReader.readEntries(async (entries) => {
						nextList = await Promise.all(
							entries.map((entry) => this.loadFileEntry(entry))
						);
						nextList = sortData(nextList);
						console.log(`文件夹有 ${nextList.length} 张图片`);
						this.$message.success(`文件夹有 ${nextList.length} 张图片`);
						this.$emit('drop', {name: firstDir.name, data: nextList});
					});
				}
			}
		},
	},
};
</script>

<style lang="scss" scoped>
.drag-and-drop {
	background: transparent;
	position: fixed;
	left: 0;
	right: 0;
	top: 0;
	bottom: 0;

	display: none;
	align-items: center;
	justify-content: center;

	// .tip {}

	&.dragging {
		display: flex;
		z-index: 100;
		background: #fff;
		opacity: 0.8;

		.tip {
			display: block;
		}
	}
}
</style>
