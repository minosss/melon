<template>
	<div id="app">
		<h1>melon</h1>
		<el-row>
			<el-col :span="14" :offset="5">
				<div
					id="drapndrop"
					class="drag-and-drop"
					@drop.prevent="onFolderDrop"
					@dragover.prevent
				>
					拖动文件夹到这里
				</div>
			</el-col>
		</el-row>
		<el-row :gutter="20">
			<el-col :span="5" :offset="2">
				<el-input v-model="previewWidth">
					<template slot="prepend">宽度</template>
				</el-input>
			</el-col>
			<el-col :span="5">
				<el-input v-model="previewPadding">
					<template slot="prepend">内边距</template>
				</el-input>
			</el-col>
			<el-col :span="5">
				<el-input v-model="columnNumber">
					<template slot="prepend">单行</template>
				</el-input>
			</el-col>
			<el-col :span="5">
				<el-input v-model="columnTop">
					<template slot="prepend">行距</template>
				</el-input>
			</el-col>
			<el-col :span="4" :offset="10">
				<el-button
					style="margin-top: 20px"
					type="success"
					@click="exportPng"
					icon="el-icon-picture"
				>
					导出文件 {{ filename }}.png
				</el-button>
			</el-col>
		</el-row>
		<el-row>
			<el-col :span="24">
				<div :style="{width: `${previewWidth}px`, margin: '0 auto'}">
					<el-divider>{{ previewWidth }}</el-divider>
				</div>
				<div v-if="list && list.length === 0">
					<p>还没选择图片</p>
				</div>
				<div
					v-else
					class="preview"
					ref="preview"
					:style="{
						width: `${previewWidth - previewPadding * 2}px`,
						padding: `${previewPadding}px`,
					}"
				>
					<div
						class="block"
						v-for="(item, index) in list"
						:key="item.name"
						:style="{margin: `${getMargin(index)}`}"
					>
						<el-image class="block-image" :src="item.src"></el-image>
						<span class="block-name">{{ item.name }}</span>
					</div>
				</div>
			</el-col>
		</el-row>
	</div>
</template>

<script>
import html2canvas from 'html2canvas';
import {saveAs} from 'file-saver';

export default {
	name: 'App',
	components: {},
	data() {
		return {
			filename: '',
			list: [],
			previewWidth: 750,
			previewPadding: 40,
			columnNumber: 4,
			columnTop: 20,
		};
	},
	computed: {
		columnSpace: function () {
			const innerWidth =
				this.previewWidth - this.previewPadding * 2 - 100 * this.columnNumber;
			return innerWidth / (this.columnNumber - 1);
		},
	},
	methods: {
		onFolderDrop(e) {
			if (e.dataTransfer.items) {
				const item = e.dataTransfer.items[0].webkitGetAsEntry();
				// 当前第一个文件夹
				if (item.isDirectory) {
					let nextList = [];
					let directoryReader = item.createReader();
					// 读取子文件
					directoryReader.readEntries(function (entries) {
						entries.forEach(function (entry) {
							if (entry.isFile) {
								entry.file((file) => {
									console.log(file);
									if (['image/jpeg', 'image/png'].includes(file.type)) {
										const reader = new FileReader(file);
										reader.onload = (e) => {
											nextList.push({
												src: e.target.result,
												name: file.name,
											});
										};
										reader.readAsDataURL(file);
									}
								});
							}
						});
					});
					this.list = nextList;
					this.filename = item.name;
				}
			}
		},
		getMargin(index) {
			let left = 0;
			let top = 0;
			if (index % this.columnNumber !== 0) {
				const innerWidth =
					this.previewWidth - this.previewPadding * 2 - 100 * this.columnNumber;
				left = innerWidth / (this.columnNumber - 1);
			}
			if (index >= this.columnNumber) {
				top = this.columnTop;
			}
			return `${top}px 0 0 ${left}px`;
		},
		// 导出图片
		exportPng() {
			if (this.list.length === 0) {
				this.$message.error('还没图片呢');
				return;
			}
			const filename = this.filename;
			html2canvas(this.$refs.preview, {backgroundColor: null}).then(function (canvas) {
				canvas.toBlob(function (blob) {
					saveAs(blob, filename + '.png');
				}, 'image/png');
			});
		},
	},
};
</script>

<style>
.el-divider {
	background: #faad14 !important;
	margin-bottom: 0 !important;
}

#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #20123b;
}

.preview {
	margin: 0 auto;
	background: transparent;
	display: flex;
	flex-wrap: wrap;
}

.block {
	display: flex;
	flex-direction: column;
}

.block-image {
	width: 100px;
	height: 100px;
}

.block-name {
	margin: 8px 0;
}

.drag-and-drop {
	margin-bottom: 20px;
	padding: 40px 0;
	background: #f6f6f6;
	border-radius: 4px;
}
</style>
