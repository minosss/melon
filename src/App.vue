<template>
	<div id="app">
		<drag-and-drop @drop="onDrop" @add="onDropAdd" />
		<!-- 侧边菜单 -->
		<div class="toolbar">
			<el-tooltip content="菜单" placement="left">
				<el-button icon="el-icon-setting" @click="showDrawer = true"></el-button>
			</el-tooltip>
			<el-tooltip :content="`导出文件 ${filename}.png`" placement="left">
				<el-button type="success" @click="exportPng" icon="el-icon-picture"></el-button>
			</el-tooltip>
			<el-tooltip content="清空" placement="left">
				<el-button type="danger" @click="clean" icon="el-icon-delete"></el-button>
			</el-tooltip>
		</div>
		<!-- 设置 -->
		<el-drawer title="MELON" :visible.sync="showDrawer">
			<div style="padding-right: 30px">
				<el-form label-width="80px">
					<el-form-item label="宽度">
						<el-input-number v-model="previewWidth"></el-input-number>
					</el-form-item>
					<el-form-item label="单行">
						<el-input-number v-model="columnNumber"></el-input-number>
					</el-form-item>
					<el-form-item label="行距">
						<el-input-number v-model="columnTop"></el-input-number>
					</el-form-item>
					<el-form-item label="图片">
						<el-row>
							<el-col :span="10">
								<el-input v-model="imageWidth"></el-input>
							</el-col>
							<el-col :span="2">
								<div class="text-center">x</div>
							</el-col>
							<el-col :span="10">
								<el-input v-model="imageHeight"></el-input>
							</el-col>
						</el-row>
					</el-form-item>
					<el-form-item label="边框">
						<el-select v-model="currentClip">
							<el-option
								v-for="path in $options.clipOptions"
								:key="path.label"
								:label="path.label"
								:value="path.value"
							/>
						</el-select>
					</el-form-item>
				</el-form>
			</div>
		</el-drawer>
		<!-- 展示 -->
		<div class="relative">
			<div :style="{width: `${previewWidth}px`}">
				<div class="text-center">
					图片 {{ list.length }} 张 / 每行 {{ columnNumber }} 张 / 行距
					{{ columnTop }} px / 图片 {{ imageWidth }}x{{ imageHeight }}
					{{ canExport ? ' / 文件名 ' + exportFilename + '.png' : '' }}
				</div>
				<el-divider>{{ previewWidth }}px</el-divider>
			</div>
			<!-- 预览 -->
			<div
				id="preview"
				class="preview"
				ref="preview"
				:style="{
					width: `${previewWidth}px`,
				}"
			>
				<svg height="0" width="0" class="clips">
					<defs>
						<clipPath id="circle2">
							<path d="M33.8,129.9a51.9,51.9,0,0,1,72.4,0,70,70,0,1,0-72.4,0 Z" />
						</clipPath>
						<clipPath id="circle" clipPathUnits="objectBoundingBox">
							<circle cx=".5" cy=".5" r=".5" />
						</clipPath>
						<clipPath id="rect" clipPathUnits="objectBoundingBox">
							<polyline
								points="0.0429 0.0000 0.0000 0.0429 0.0429 0.0857 0.0000 0.1357 0.0429 0.1786 0.0000 0.2214 0.0429 0.2714 0.0000 0.3143 0.0429 0.3571 0.0000 0.4071 0.0429 0.4500 0.0000 0.5000 0.0429 0.5429 0.0000 0.5857 0.0429 0.6357 0.0000 0.6786 0.0429 0.7214 0.0000 0.7714 0.0429 0.8143 0.0000 0.8571 0.0429 0.9071 0.0000 0.9500 0.0429 1.0000 0.0857 0.9500 0.1357 1.0000 0.1786 0.9500 0.2214 1.0000 0.2714 0.9500 0.3143 1.0000 0.3571 0.9500 0.4071 1.0000 0.4500 0.9500 0.5000 1.0000 0.5429 0.9500 0.5857 1.0000 0.6357 0.9500 0.6786 1.0000 0.7214 0.9500 0.7714 1.0000 0.8143 0.9500 0.8571 1.0000 0.9071 0.9500 0.9500 1.0000 1.0000 0.9500 0.9500 0.9071 1.0000 0.8571 0.9500 0.8143 1.0000 0.7714 0.9500 0.7214 1.0000 0.6786 0.9500 0.6357 1.0000 0.5857 0.9500 0.5429 1.0000 0.5000 0.9500 0.4500 1.0000 0.4071 0.9500 0.3571 1.0000 0.3143 0.9500 0.2714 1.0000 0.2214 0.9500 0.1786 1.0000 0.1357 0.9500 0.0857 1.0000 0.0429 0.9500 0.0000 0.9071 0.0429 0.8571 0.0000 0.8143 0.0429 0.7714 0.0000 0.7214 0.0429 0.6786 0.0000 0.6357 0.0429 0.5857 0.0000 0.5429 0.0429 0.5000 0.0000 0.4500 0.0429 0.4071 0.0000 0.3571 0.0429 0.3143 0.0000 0.2714 0.0429 0.2214 0.0000 0.1786 0.0429 0.1357 0.0000 0.0857 0.0429 0.0429 0.0000"
							/>
						</clipPath>
						<clipPath id="diamond" clipPathUnits="objectBoundingBox">
							<polyline
								points="0.5211 0.0000 0.4737 0.0000 0.4737 0.0474 0.4263 0.0474 0.4263 0.0947 0.3789 0.0947 0.3789 0.1421 0.3316 0.1421 0.3316 0.1895 0.2842 0.1895 0.2842 0.2368 0.2368 0.2368 0.2368 0.2842 0.1895 0.2842 0.1895 0.3316 0.1421 0.3316 0.1421 0.3789 0.0947 0.3789 0.0947 0.4263 0.0474 0.4263 0.0474 0.4737 0.0000 0.4737 0.0000 0.5211 0.0474 0.5211 0.0474 0.5684 0.0947 0.5684 0.0947 0.6158 0.1421 0.6158 0.1421 0.6632 0.1895 0.6632 0.1895 0.7105 0.2368 0.7105 0.2368 0.7579 0.2842 0.7579 0.2842 0.8053 0.3316 0.8053 0.3316 0.8526 0.3789 0.8526 0.3789 0.9000 0.4263 0.9000 0.4263 0.9526 0.4737 0.9526 0.4737 1.0000 0.5211 1.0000 0.5211 0.9526 0.5684 0.9526 0.5684 0.9000 0.6158 0.9000 0.6158 0.8526 0.6632 0.8526 0.6632 0.8053 0.7105 0.8053 0.7105 0.7579 0.7579 0.7579 0.7579 0.7105 0.8053 0.7105 0.8053 0.6632 0.8526 0.6632 0.8526 0.6158 0.9000 0.6158 0.9000 0.5684 0.9526 0.5684 0.9526 0.5211 1.0000 0.5211 1.0000 0.4737 0.9526 0.4737 0.9526 0.4263 0.9000 0.4263 0.9000 0.3789 0.8526 0.3789 0.8526 0.3316 0.8053 0.3316 0.8053 0.2842 0.7579 0.2842 0.7579 0.2368 0.7105 0.2368 0.7105 0.1895 0.6632 0.1895 0.6632 0.1421 0.6158 0.1421 0.6158 0.0947 0.5684 0.0947 0.5684 0.0474 0.5211 0.0474 0.5211 0.0000"
							/>
						</clipPath>
						<clipPath id="polygon" clipPathUnits="objectBoundingBox">
							<polygon
								points="0.2850 0.0616 0.3200 0.0137 0.3550 0.0616 0.3900 0.0137 0.4250 0.0616 0.4600 0.0137 0.5000 0.0616 0.5350 0.0137 0.5700 0.0616 0.6050 0.0137 0.6400 0.0616 0.6750 0.0137 0.7100 0.0616 0.7450 0.0137 0.7800 0.0616 0.8150 0.0137 0.8500 0.0616 0.8850 0.0137 0.9200 0.0616 0.9550 0.0137 0.9950 0.0616 0.9550 0.1096 0.9950 0.1575 0.9550 0.2055 0.9950 0.2534 0.9550 0.3014 0.9950 0.3493 0.9550 0.3973 0.9950 0.4452 0.9550 0.5000 0.9950 0.5479 0.9550 0.5959 0.9950 0.6438 0.9550 0.6918 0.9950 0.7397 0.9550 0.7877 0.9950 0.8356 0.9550 0.8836 0.9950 0.9315 0.9550 0.9795 0.9200 0.9315 0.8850 0.9795 0.8500 0.9315 0.8150 0.9795 0.7800 0.9315 0.7450 0.9795 0.7100 0.9315 0.6750 0.9795 0.6400 0.9315 0.6050 0.9795 0.5700 0.9315 0.5350 0.9795 0.5000 0.9315 0.4600 0.9795 0.4250 0.9315 0.3900 0.9795 0.3550 0.9315 0.3200 0.9795 0.2850 0.9315 0.2500 0.9795 0.2150 0.9315 0.1800 0.9795 0.1450 0.9315 0.1100 0.9795 0.0750 0.9315 0.0400 0.9795 0.0050 0.9315 0.0400 0.8836 0.0050 0.8356 0.0400 0.7877 0.0050 0.7397 0.0400 0.6918 0.0050 0.6438 0.0400 0.5959 0.0050 0.5479 0.0400 0.5000 0.0050 0.4452 0.0400 0.3973 0.0050 0.3493 0.0400 0.3014 0.0050 0.2534 0.0400 0.2055 0.0050 0.1575 0.0400 0.1096 0.0050 0.0616 0.0400 0.0137 0.0750 0.0616 0.1100 0.0137 0.1450 0.0616 0.1800 0.0137 0.2150 0.0616 0.2500 0.0137 0.2850 0.0616"
							/>
						</clipPath>
					</defs>
				</svg>
				<!--  -->
				<div class="text-center" v-if="list && list.length === 0">
					<p>还没选择图片</p>
				</div>
				<draggable
					v-else
					class="preview-content"
					v-model="list"
					v-bind="dragOptions"
					:style="contentStyle"
				>
					<card
						v-for="(item, index) in displayList"
						:key="`${item.name}_${index}`"
						:data="item"
						:width="imageWidth"
						:height="imageHeight"
						:clipClass="currentClip"
						@remove="onRemove"
					/>
				</draggable>
			</div>
		</div>
	</div>
</template>

<script>
import draggable from 'vuedraggable';
import * as html2image from 'html-to-image';
import {saveAs} from 'file-saver';
import Card from './components/Card';
import DragAndDrop from './components/DragAndDrop';

//-
const clipOptions = [
	{
		label: '无',
		value: 'clip-none',
	},
	{
		label: '圆●',
		value: 'clip-circle',
	},
	{
		label: '正方形■',
		value: 'clip-rect',
	},
	{
		label: '长方形200x146',
		value: 'clip-polygon',
	},
	{
		label: '菱形◆',
		value: 'clip-diamond',
	},
];
//-

export default {
	name: 'App',
	components: {
		Card,
		DragAndDrop,
		draggable,
	},
	clipOptions,
	data() {
		return {
			// export filename
			filename: '',
			list: [],
			currentClip: 'clip-none',
			// export width
			previewWidth: 1080,
			// column
			columnNumber: 3,
			columnTop: 40,
			// show/hidden settings
			showDrawer: false,
			// w * h
			imageWidth: 300,
			imageHeight: 186,
			// image (total / column) / split
			splitRow: 14,
			split: false,
		};
	},
	computed: {
		dragOptions() {
			return {
				animation: 150,
			};
		},
		contentStyle() {
			return {
				display: 'grid',
				'justify-content': 'center',
				'grid-gap': `${this.columnTop}px`,
				'grid-template-columns': `repeat(${this.columnNumber}, ${this.imageWidth}px)`,
			};
		},
		displayList() {
			return this.split ? [] : this.list;
		},
		exportFilename() {
			return this.filename || 'mixed';
		},
		canExport() {
			return this.list && this.list.length > 0;
		},
	},
	methods: {
		clean() {
			this.list = [];
			this.filename = '';
		},
		onDrop(e) {
			// if (e.data.length > 80) {
			// 	this.$confirm('超过 80 个图片，是否使用分块？', '提示', {})
			// 		.then(() => {
			// 			this.list = e.data;
			// 			this.filename = e.name;
			// 			this.split = true;
			// 		})
			// 		.catch(() => {
			// 			this.list = e.data;
			// 			this.filename = e.name;
			// 			this.split = false;
			// 		});
			// } else {
			this.list = e.data;
			this.filename = e.name;
			this.split = false;
			// }
		},
		onDropAdd(e) {
			this.list = [...this.list, ...e.data];
		},
		onRemove(targetName) {
			this.list = this.list.filter((item) => item.name !== targetName);
		},
		// 导出图片
		exportPng() {
			if (!this.canExport) {
				this.$message.error('还没图片呢');
				return;
			}
			const filename = this.exportFilename;
			html2image
				.toBlob(document.getElementById('preview'), {pixelRatio: 1})
				.then((blob) => {
					saveAs(blob, filename + '.png');
				});
		},
	},
};
</script>

<style lang="scss" src="./App.scss"></style>
