<template>
	<div>
		<el-form>
			<!--<el-form-item :label="(qindex+1)+taccord+item.title+':'" @mouseover.native.prevent="showcart(item)" @mouseout.native.prevent="showcart(item)" :class="{'bordernone':item.edittextinput,'itemborder':item.show,'itemmust':item.is_must}">-->
				<el-form-item :label="item.qtitle+taccord+item.title+':'" @mouseover.native.prevent="showcart(item)" @mouseout.native.prevent="showcart(item)" :class="{'bordernone':item.edittextinput,'itemborder':item.show,'itemmust':item.is_must}">					
				<!--<i v-if="item.is_must" v-text="'*'" class="itemmust"></i>-->
				<el-radio-group v-model="item.default_choose" @change="dochange">
					<el-radio :label="im.option_name" v-for="(im,inx) in item.option" :key="inx" class="singradio">{{im.option_name}}</el-radio>
				</el-radio-group>
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
				<el-row v-if="item.edittextinput" class="edittextinput">
					<el-col class="singleinputcontent">
						<el-form-item :label="'题目文本'">
							<el-input v-model="item.title"></el-input>
							<el-checkbox label="必答" name="type" v-model="item.is_must" :disabled="status!='1' && type!='0'"></el-checkbox>
							<div class="singleedit">
								<el-row type="flex">
									<el-col :span="12">项目编辑:</el-col>
									<el-col :span="4">分值</el-col>
									<el-col :span="3">默认</el-col>
									<el-col :span="5" style="text-align: center;">操作</el-col>
								</el-row>
								<!--	<el-form-item v-for="(domain, index) in item.domains" :label="'域名' + index" :key="domain.key" :prop="'domains.' + index + '.value'" :rules="{required: true, message: '域名不能为空', trigger: 'blur'}">-->

								<el-form-item v-for="(domain, dindex) in item.option" :rules="{required: true, message: '域名不能为空', trigger: 'blur'}" :key="dindex">
									<el-row>
										<el-col :span="12" class="option_name">
											<el-input v-model="domain.option_name"></el-input>
										</el-col>
										<el-col :span="4" class="grade">
											<el-input v-model="domain.score"></el-input>
										</el-col>
										<el-col :span="3">
											<!--<el-radio-group v-model="item.default_choose" @change="gdochange">-->
											<el-radio-group v-model="item.default_choose">
												<el-radio :label="domain.option_name" @click.native.prevent="gdochange(domain.option_name)"></el-radio>
											</el-radio-group>
										</el-col>
										<el-col :span="5" class="iconplus">
											<i class="el-icon-circle-plus-outline" @click="addDomain"></i>
											<i class="el-icon-remove-outline" @click="removeDomainitem(index,qindex,dindex)"></i>
											<i :class="{radioDisable:(dindex+1)==item.option.length}" class="el-icon-back" @click="domainSortdown(index,qindex,dindex,'down')"></i>
											<i :class="{radioDisable:dindex==0}" class="el-icon-back backright" @click="domainSortdown(index,qindex,dindex,'up')"></i>
										</el-col>
									</el-row>
								</el-form-item>

								<div class="btngroup">
									<el-button @click="addDomain" type="primary" plain>+新增选项</el-button>
									<el-button @click="relevance(item)" type="primary" plain>+关联逻辑</el-button>
									<el-button @click="jump" type="primary" plain>+跳转逻辑</el-button>
									<jump :jumpshow='jumpshow' :domains="item.option" :item="item" @canclejump='canclejump' :qlist="qlist" @surejump="surejump"></jump>
									<relevance :relevanceshow='relevanceshow' :qlist="qlist" :relatetype="relatetype" :list="list" :item="item" @canclerelevance='canclerelevance' :index="index" :qindex="qindex" @surerelevance="surerelevance"></relevance>
		
								</div>

							</div>
							<p class="tips">注：关联逻辑与跳转逻辑只能设置其中一项</p>
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
	import bus from './eventBus';
	import jump from './jump.vue';
	import relevance from './relevance.vue';
	export default {
		data() {
			return {
				poSition: '',
				jumpshow: false,
				relevanceshow: false,
			}
		},
		props: {
			item: {
				type: Object,
				default: {}
			},
			index: {
				type: Number,
				default: 0
			},
			qindex: {
				type: Number,
				default: 0
			},
			qlist: {
				type: Array,
				default: () => []
			},
			list: {
				type: Array,
				default: () => []
			},
			taccord: {
				type: String,
				default: ""
			},
			relatetype: {
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
		methods: {
			showedit(item) {
				if(this.status != "1" && !this.$route.query.templateId) {
					return;
				}

				item.edittextinput = !item.edittextinput;
			},
			dochange(item) {

				//				this.gdomack = item;
			},
			gdochange(item) {

				if(this.status != "1" && !this.$route.query.templateId) {
					return;
				}
				this.$emit("changeDomainRadio", this.index, this.qindex, item);

			},
			showcart(item) {
				//console.log(this.$route.query.templateId)
				if(this.status != "1" && !this.$route.query.templateId) {
					return;
				}
				//				debugger
				//				console.log(item.show);
				item.show = item.edittextinput || !item.show;
				//				console.log(item.show);
			},
			submitForm(item) {

				if(this.status != "1" && !this.$route.query.templateId) {
					return;
				}
				this.$emit("submitForm", item, this.index);
			},
			removeDomain() {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("removeDomain", this.index, this.qindex);
			},
			itemSortdown: function(item, index, qindex, type) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("itemSortdown", item, index, qindex, type);
				item.show = false;
				item.edittextinput = false;
			},
			domainSortdown: function(index, qindex, dindex, type) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}!event.target.classList.contains("radioDisable") && this.$emit("domainSortdown", index, qindex, dindex, type)
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
			addDomain() {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("addDomain", this.index, this.qindex);
			},
			removeDomainitem(index, qindex, dindex) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("removeDomainitem", index, qindex, dindex);
			},
			changeposition(item) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				item.changeButton = !item.changeButton;
			},
			jump(item) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}

				//
				//				debugger
				//				for(var i = 0; i < item.option.length; i++) {
				//					if(item.option[i].skip_sub != "") {
				//						var skipList = this.qlist[item.option[i].skip_sub];
				//						item.option[i].skip_subName = (parseInt(item.option[i].skip_sub) + 1) + "" + skipList.title;
				//					} else {
				//						item.option[i].skip_subName = "请选择";
				//					}
				//				}
				this.jumpshow = true;
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
			canclejump(item) {
				for(let i in item.option) {
					item.option[i].skip_sub = "";
				}
				this.jumpshow = false;
			},
			canclerelevance(item) {
				for(let i in item.option) {
					item.option[i].related_sub = "";
				}
				item.relevanceshow = false;
					this.item.show=true;
				this.item.edittextinput = true;
			},
			surejump() {
				this.jumpshow = false;
					this.item.show=true;
				this.item.edittextinput = true;
			},
			surerelevance(item) {
				item.relevanceshow = false;
				
				this.item.show=true;
				this.item.edittextinput = true;
			}

		},
		components: {
			headTop,
			jump,
			relevance
		},
		created() {

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
		width: 18%;
		margin-left: 0;
	}
	
	.singleedit .el-radio__input.is-checked+.el-radio__label {
		display: none;
	}
	
	.singleedit .el-radio__label {
		display: none;
	}
	
	.radioDisable {
		color: #D9D9D9;
		border-color: #D9D9D9 !important;
	}
	
	.singleinputcontent .el-input__inner {
		height: 34px;
		line-height: 34px;
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
	
	.topic .el-form>.el-form-item {
		    margin: 0 auto;
    width: 90%;
    padding: 10px 10px 0;
		border: 1px solid transparent;
		margin-bottom: 5px;
		padding-bottom: 40px;
	}
	
	.el-form-item__label {}
	
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
		padding: 10px 0;
		margin-top: 20px;
		display: block;
	}
	
	.el-form-item .edittextinput {
		border: 1px dashed rgb(121, 121, 121);
		height: auto;
		padding: 10px 5%;
	}
	
	.el-form-item .el-checkbox {
		margin-left: 10%;
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
	
	.singleinputcontent p {
		width: 70%;
		padding-left: 5px;
		color: red;
		background: rgb(242, 242, 242);
		margin-top: 20px;
	}
	
	.singleinputcontent .el-input {
		margin-left: 0;
	}
	
	.singleedit .el-row {
		margin-bottom: 10px;
		>.el-col {
			text-align: center;
			&:nth-of-type(1) {
				text-align: left;
			}
		}
	}
	
	.singleedit {
		.iconplus {
			display: flex;
			align-items: center;
			justify-content: flex-start;
			height: 40px;
			padding-left: 30px;
		}
		i {
			font-size: 24px;
			align-items: center;
			margin-right: 5px;
		}
		.el-radio {
			color: rgba(255, 255, 255, 0);
			border: none;
		}
		.el-input {
			width: 60%;
		}
		.el-icon-back {
			display: flex;
			transform: rotate(-90deg);
			border: 1px solid #303133;
			border-radius: 50%;
			font-weight: bold;
			padding: 2px;
			height: 15px;
			width: 15px;
			font-size: 14px;
		}
		.backright {
			transform: rotate(90deg);
		}
		.btngroup {
			display: flex;
			justify-content: space-between;
			.el-button {
				width: 25%;
			}
		}
	}
	
	.positionsure {
		width: 40px;
		display: inline-block;
		margin: 0;
		padding: 3px 0;
	}
	
	.singleedit .option_name .el-input {
		width: 90%;
	}
	/*.singleedit .el-row > .el-col.grade{
		text-align: left;
	}*/
	
	.singradio {
		width: 100%;
		margin: 10px 0;
	}
	
	.singradio+.singradio {
		margin-left: 0;
	}
</style>