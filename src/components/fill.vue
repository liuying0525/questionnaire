<template>
	<div>
		<el-form class="fillcontent">
			<!--v-for="(item,index) in formlist" :key="index"-->
			<el-form-item :label="item.qtitle+taccord+item.title+':'" @mouseover.native.prevent="showcart(item)" @mouseout.native.prevent="showcart(item)" :class="{'bordernone':item.edittextinput,'itemborder':item.show,'itemmust':item.is_must}">
				<!--<i v-if="item.is_must" v-text="'*'" class="itemmust"></i>-->
				<el-input></el-input>
				<div v-show="item.show" class="transition-box">
					<span @click.stop.prevent="showedit(item)">编辑</span>
					<span @click.stop.prevent="removeDomain(index,qindex)">删除</span>
					<span @click.stop.prevent="changeposition(item)">位置变更</span>
					<div class="changeposition" v-if="item.changeButton">
						<el-button type="info" plain @click.stop.prevent="itemSortdown(item,index,qindex,'up')">上移一题</el-button>
						<el-button type="info" plain @click.stop.prevent="itemSortdown(item,index,qindex,'down')">下移一题</el-button>
						<div>移至【
							<el-input v-model="item.poSition" class="inputposition"></el-input>】题
							<el-button type="primary" plain class="positionsure" @click.stop.prevent="itemSortdown(item,index,qindex,'jumpitem')">确定</el-button>
						</div>
					</div>
				</div>
				<el-row v-if="item.edittextinput">
					<el-col>
						<el-form-item :label="'题目文本'">
							<el-input v-model="item.title"></el-input>
							<el-checkbox label="必答" name="type" v-model="item.is_must" :disabled="status!='1' && type!='0'"></el-checkbox>
							<div class="btngroup">
								<el-button type="primary" plain disabled>+新增选项</el-button>
								<el-button @click="relevance(item)" type="primary" plain>+关联逻辑</el-button>
								<el-button type="primary" plain disabled>+跳转逻辑</el-button>
								<relevance :relevanceshow='relevanceshow' :qlist="qlist" :relatetype="relatetype" :list="list" :item="item" @canclerelevance='canclerelevance' :index="index" :qindex="qindex" @surerelevance="surerelevance"></relevance>
							</div>
							<el-button type="primary" @click="submitForm(item)">保存</el-button>
						</el-form-item>
					</el-col>
				</el-row>
			</el-form-item>

		</el-form>

	</div>

</template>

<script>
	import headTop from 'view/head/headTop.vue';
	import relevance from './relevance.vue';
	
	export default {
		data() {
			return {
				poSition: '',
				relevanceshow: false
			}
		},
		props: {
			item: {
				type: Object,
				default: {}
			},
			relatetype: {
				type: String,
				default: ""
			},
			index: {
				type: Number,
				default: 0
			},
			qindex: {
				type: Number,
				default: 0
			},
			list: {
				type: Array,
				default: () => []
			},
			qlist: {
				type: Array,
				default: () => []
			},
			taccord: {
				type: String,
				default: ""
			},
			status: {
				type: String,
				default: "1"
			},
			type: {
				type: String,
				default: ""
			}
		},
		created() {
		},
		methods: {
			showedit(item) {
				if(this.status != "1") {
					return;
				}
				item.edittextinput = !item.edittextinput;
			},
			showcart(item) {
				if(this.status != "1") {
					return;
				}
				//				console.log(item.show)

				//				item.show = item.edittextinput || !item.show;
				item.show = item.edittextinput || !item.show
				//				console.log(item.show)

			},
			submitForm(item) {
				if(this.status != "1") {
					return;
				}
				this.$emit("submitForm", item, this.index);
			},
			itemSortdown: function(item, index, qindex, type) {
				this.$emit("itemSortdown", item, index, qindex, type);
				item.show = false;
				item.edittextinput = false;
				//				item.show=false;
			},
			removeDomain(item) {
				if(this.status != "1") {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("removeDomain", this.index, this.qindex);
			},
			command(callback, vc) {
				console.log("回调参数" + callback);
				if(!callback) {
					var ctx = this;
					ctx.AREACODE2 = '请选择';
					if(vc != '') {
						ctx.show2 = true;
						ctx.getAreaListDataSearch(vc, 1);
					}
				}
			},
			relevance(item) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				item.relevanceshow = true;
			},
			surerelevance(item) {
				item.relevanceshow = false;
//					this.item.show=true;
//				this.item.edittextinput = true;
			},
			canclerelevance(item) {
				item.relevanceshow = false;
//					this.item.show=true;
//				this.item.edittextinput = true;
			},
			changeposition(item) {
				if(this.status != "1") {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				item.changeButton = !item.changeButton;
				
			}
		},
		components: {
			headTop,
			relevance
		}
	}
</script>
<style>
	.changeposition .inputposition .el-input__inner {
		background-color: rgb(245, 245, 245);
		height: 14px;
		line-height: 14px;
		margin: 0;
		border: none;
		padding: 0 5px;
	}
	
	.el-form-item .el-input.inputposition {
		margin-left: 0;
	}
	
	.topic .fillcontent .el-form-item__label {
		width: auto;
	}
	
	.topic .fillcontent .el-form-item__content .el-input {
		margin-left: 10px;
	}
</style>
<style scoped="scoped" lang="scss">
	.el-input {
		width: 30%;
		margin-left: 76px;
	}
	
	.textinputhidden {
		display: none;
	}
	
	.el-form-item__label {}
	
		.topic .el-form>.el-form-item {
		    margin: 0 auto;
    width: 90%;
    padding: 10px 10px 0;
		border: 1px solid transparent;
		margin-bottom: 5px;
		padding-bottom: 40px;
	}
	
	.el-form>.el-form-item.itemborder {
		border: 1px solid #eee;
		padding-bottom: 0;
	}
	
	.el-form>.el-form-item.bordernone {
		border: 1px solid transparent;
		height: auto;
		/*margin-bottom:30px;*/
		padding-bottom: 40px;
	}
	
	.el-button {
		width: 100%;
		padding: 5px 0;
		margin-top: 20px;
		display: block;
	}
	
	.el-form-item .el-form-item {
		border: 1px dashed rgb(121, 121, 121);
		height: auto;
		padding: 10px 5%;
	}
	

	.el-form-item .el-checkbox {
		margin-left: 10%;
		position: relative;
	}
	
	.el-form-item .el-input {
		height: 32px;
		line-height: 32px;
	}
	
	.el-collapse-item {
		padding-bottom: 20px;
	}
	
	.transition-box span {
		color: rgb(41, 155, 252);
		margin-right: 15%;
		cursor: pointer;
		display: inline-block;
		width: 80px;
	}
	
	.el-dropdown-menu {
		/*position:static !important;*/
	}
	
	.changeposition {
		padding: 5px 0;
		width: 20%;
		position: absolute;
		right: 23%;
		z-index: 1000;
		top: 5%;
		background: rgb(245, 245, 245);
		text-align: center;
		>* {
			color: #000;
			height: 24px;
			line-height: 24px;
			margin: 0;
			padding: 0;
		}
	}
	
	.changeposition .el-button {
		border: none;
	}
	
	.changeposition>div>span {
		display: inline-block;
		width: 0;
	}
	
	.el-button+.el-button {
		margin-left: 0;
	}
	
	.el-form-item__content {
		position: relative;
	}
</style>