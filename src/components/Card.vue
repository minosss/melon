<template>
	<div class="block" v-loading="loading">
		<span class="remove" @click="onClick">
			<!-- eslint-disable-next-line prettier/prettier -->
			<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-x-circle"><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg>
		</span>
		<el-image
			fit="cover"
			:style="{width: `${width}px`, height: `${height}px`}"
			:class="{'block-image': true, [clipClass]: !!clipClass}"
			:src="image"
		/>
		<el-input
			ref="nameInput"
			class="block-name-editor"
			v-if="editName"
			v-model="name"
			@blur="onExitEditName"
		/>
		<span v-else class="block-name" @dblclick="tryEditName">{{ name }}</span>
	</div>
</template>

<script>
export default {
	name: 'card',
	props: {
		// image: String,
		// name: String,
		data: {
			type: Object,
			required: true,
		},
		clipClass: String,
		clip: {
			type: String,
			default: 'none',
		},
		width: {
			type: [Number, String],
			default: 100,
		},
		height: {
			type: [Number, String],
			default: 100,
		},
	},
	data: function () {
		return {
			loading: false,
			name: '',
			// 1x1
			image: 'data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==',
			editName: false,
		};
	},
	mounted() {
		this.loadImage();
	},
	methods: {
		onClick() {
			this.$emit('remove', this.name);
		},
		loadImage() {
			const {file, name} = this.data;
			this.loading = true;
			this.name = name.replace(/\.(jpg|png|jpeg)/, '');
			console.log('加载 ', name);
			const reader = new FileReader(file);
			reader.onload = (e) => {
				console.log('完成 ', file.name);
				this.loading = false;
				this.image = e.target.result;
			};
			reader.readAsDataURL(file);
		},
		tryEditName() {
			this.editName = true;
			this.$nextTick(() => {
				this.$refs.nameInput.focus();
			});
		},
		onExitEditName() {
			this.editName = false;
		},
	},
};
</script>

<style lang="scss" scoped>
.block {
	position: relative;
	display: inline-flex;
	flex-direction: column;

	&:hover {
		.remove {
			display: inline-flex;
		}
	}
}

.remove {
	cursor: pointer;
	z-index: 100;
	background: white;
	border-radius: 50%;
	overflow: hidden;
	padding: 4px;
	display: none;
	position: absolute;
	right: 0;
	top: 0;
}

// .block-image {}

.block-name {
	font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
		'Open Sans', 'Helvetica Neue', sans-serif;
	text-align: center;
	font-size: 28px;
	font-weight: 500;
	margin: 0;
	margin-top: 8px;
}
</style>

<style lang="scss">
.block-name-editor {
	.el-input__inner {
		text-align: center;
		font-size: 20px;
	}
}
</style>
