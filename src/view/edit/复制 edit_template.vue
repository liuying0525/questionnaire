<template>
	<div class="edit_tempbg">
		<el-row :gutter="20" class="top">
			<el-col :span="3" :offset="9">
				<i class="el-icon-search"></i>预览</el-col>
			<el-col :span="3" @click.native="finishSub()"><i class="el-icon-check"></i>完成</el-col>
		</el-row>
		<div class="editTemContain">
			<div>
				<el-row type="flex" justify="end" class="conTop">
					<el-col :span="3" @click.native="openModel"><i class="el-icon-plus"></i>新建模块</el-col>
				</el-row>
				<div class="conBottom">
					<div class="conBottomT">
						<el-input type="text" v-model="questiontitle" placeholder="问卷标题" class="questiontitle"></el-input>
						<el-input type="textarea" v-model="contentText" placeholder="问卷说明" class="questiontitle"></el-input>
					</div>

					<el-collapse v-model="activeNames" @change="handleChange">
						<div class="edit_item" v-for="(item,index) in list" :key="index">
							<el-dropdown placement="bottom">
								<span class="el-dropdown-link">
						        	<i class="new"></i> 新建题目
						        </span>
								<el-dropdown-menu slot="dropdown" class="topicdropdown">
									<el-dropdown-item @click.native="addItem(index,'fill')">填空题</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'single')">选择题</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'multiple')">多选题</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'loCation')">位置上传</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'uploadimg')">图片上传</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'multistage')">多级下拉</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'fractions')">分数题</el-dropdown-item>
									<el-dropdown-item @click.native="addItem(index,'comprehensive')">综合题</el-dropdown-item>
								</el-dropdown-menu>
							</el-dropdown>
							<el-input v-model="item.mod_name" placeholder="模块名称" class="titlename"></el-input>
							<el-collapse-item :title="item.qtitle" :name="index">

								<div class="topic" v-for="(qitem,qindex) in item.qlist" :key="qindex">

									<template v-if="qitem.sub_cat=='fill'">
										<fill :item="qitem" @removeDomain="removeDomain" :taccord="taccord" :index="index" :qindex="qindex" @itemSortdown="itemSortdown" @submitForm="submitForm"></fill>
									</template>
									<template v-if="qitem.sub_cat=='single'">
										<single :item="qitem" :qlist="item.qlist" :taccord="taccord" @changeDomainRadio="changeDomainRadio" @addDomain="addDomain" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm"></single>
									</template>
									<template v-if="qitem.sub_cat=='multiple'">
										<multiple :item="qitem" @addDomain="addDomain" :taccord="taccord" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm"></multiple>
									</template>
									<template v-if="qitem.sub_cat=='multistage'">
										<multistage :item="qitem" :taccord="taccord" :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm"></multistage>
									</template>
									<template v-if="qitem.sub_cat=='loCation'">
										<loCation :item="qitem" :taccord="taccord" :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm"></loCation>
									</template>
									<template v-if="qitem.sub_cat=='uploadimg'">
										<uploadimg :item="qitem" :taccord="taccord" :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm"></uploadimg>
									</template>
									<template v-if="qitem.sub_cat=='fractions'">
										<fractions :item="qitem" :taccord="taccord" :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm"></fractions>
									</template>
									<template v-if="qitem.sub_cat=='comprehensive'">
										<comprehensive @deleCom="delem" :comtaccord="comtaccord" :index="index" :comitem="qitem" :qindex="qindex" @itemSortdown="itemSortdown"></comprehensive>
									</template>
								</div>
							</el-collapse-item>
						</div>
					</el-collapse>
				</div>
			</div>
		</div>

	</div>
</template>

<script>
	
	import topic from 'components/topic.vue';
	import fill from 'components/fill.vue';
	import single from 'components/single.vue';
	import multiple from 'components/multiple.vue';
	import multistage from 'components/multistage.vue';
	import uploadimg from 'components/uploadimg.vue';
	import loCation from 'components/loCation.vue';
	import fractions from 'components/fractions.vue';
	import { Message } from "element-ui";
	import comprehensive from 'components/comprehensive.vue';
	import { ofill, osingle, omultiple, omultistage, ouploadimg, oloCation, ofractions } from "components/itemType";

	export default {
		data() {
			return {
				contentText: '',
				questiontitle: '',
				activeNames: [],
				list: [],
				taccord: "、",
				comtaccord: "、",
				subId: "",
				parentModle: 0
			}
		},
		methods: {
			handleChange(val) {
				console.log(val);
			},
			addfill(index) {
				let ix = this.list[index].qlist.length + 1;
				let ifill = JSON.parse(JSON.stringify(ofill));
				ifill.ppid = this.subId;
				ifill.pid = this.list[index].id;
				ifill.serial_number = ix;
				ifill.qtitle = ix;
				this.list[index].qlist.push(ifill);
			},
			addsingle(index) {
				let ix = this.list[index].qlist.length + 1;
				let isingle = JSON.parse(JSON.stringify(osingle));
				isingle.ppid = this.subId;
				isingle.pid = this.list[index].id;
				isingle.serial_number = ix;
				isingle.qtitle = ix;
				this.list[index].qlist.push(isingle);
			},
			addmultiple(index) {
				let ix = this.list[index].qlist.length + 1;
				let imultiple = JSON.parse(JSON.stringify(omultiple));
				imultiple.ppid = this.subId;
				imultiple.pid = this.list[index].id;
				imultiple.serial_number = ix;
				imultiple.qtitle = ix;
				this.list[index].qlist.push(imultiple);

			},
			addmultistage(index) {
				let ix = this.list[index].qlist.length + 1;
				let imultistage = JSON.parse(JSON.stringify(omultistage));
				imultistage.ppid = this.subId;
				imultistage.pid = this.list[index].id;
				imultistage.serial_number = ix;
				imultistage.qtitle = ix;
				this.list[index].qlist.push(imultistage);
			},
			adduploadimg(index) {
				let ix = this.list[index].qlist.length + 1;
				let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
				iuploadimg.ppid = this.subId;
				iuploadimg.pid = this.list[index].id;
				iuploadimg.serial_number = ix;
				iuploadimg.qtitle = ix;
				this.list[index].qlist.push(iuploadimg);
			},
			addloCation(index) {
				let ix = this.list[index].qlist.length + 1;
				let iloCation = JSON.parse(JSON.stringify(oloCation));
				iloCation.ppid = this.subId;
				iloCation.pid = this.list[index].id;
				iloCation.serial_number = ix;
				iloCation.qtitle = ix;
				this.list[index].qlist.push(iloCation);
			},
			addfractions(index) {
				let ix = this.list[index].qlist.length + 1;
				let ifractions = JSON.parse(JSON.stringify(ofractions));
				ifractions.ppid = this.subId;
				ifractions.pid = this.list[index].id;
				ifractions.serial_number = ix;
				ifractions.qtitle = ix;
				this.list[index].qlist.push(ifractions);
			},
			addcomprehensive(index) {
				let ix = this.list[index].qlist.length + 1;
				var option = {};
				option.title = "综合题名称";
				option.qtitle = this.list[index].qlist.length + 1;
				option.qlist = [];
				option.changeButton = false;
				option.serial_number = ix;
				option.id = 0;
				option.ppid = this.subId;
				option.pid = this.list[index].id;
				option.sub_cat = "comprehensive";
				option.poSition = "";
				debugger
				this.$post("/Home/Subject/createNewMod", {
					pid: this.subId,
					chief: option.pid,
					mod_name: option.title
				}).then((res) => {
					option.id = res.id;
					this.list[index].qlist.push(option);
				});

			},
			delem(index, pindex) {
				this.list[index].qlist.splice(pindex, 1);
			},
			addItem(index, type) {

				switch(type) {
					case "fill":
						{
							this.addfill(index);
						}
						break;
					case "single":
						{
							this.addsingle(index);
						}
						break;
					case "multiple":
						{
							this.addmultiple(index);
						}
						break;
					case "multistage":
						{
							this.addmultistage(index);
						}
						break;
					case "uploadimg":
						{
							this.adduploadimg(index);
						}
						break;
					case "loCation":
						{
							this.addloCation(index);
						}
						break;
					case "fractions":
						{
							this.addfractions(index);
						}
						break;
					case "comprehensive":
						{
							this.addcomprehensive(index);
						}
						break;
					default:
						break;
				}
				this.activeNames.indexOf(index) == -1 && this.activeNames.push(index)
			},
			addDomain(index, qindex) {
				let sort = this.list[index].qlist[qindex].option.length + 1;
				let options = {
					id: 0,
					order_num: sort,
					option_name: "选项" + sort,
					default_choose: 0,
					related_sub: '',
					skip_sub: ''
				}
				this.list[index].qlist[qindex].option.push(options);
			},
			changeDomainRadio(index, qindex, v) {
				this.list[index].qlist[qindex].default_choose = v;
				let domainlist = this.list[index].qlist[qindex].option;
				for(let i in domainlist) {
					if(domainlist[i].option_name == v) {
						domainlist[i].default_choose = 1;
					}
				}
			},
			removeDomain(index, qindex) {
				this.$confirm('您确定要删除吗?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					let dmodel = this.list[index].qlist[qindex];
					debugger
					if(dmodel.id != 0) {
						this.$post("/Home/Subject/deleteItem", {
							id: dmodel.id
						}).then((res) => {});
					}
					let nlist = this.list[index].qlist.deleteIndex(qindex);
					this.list[index].qlist = nlist;
				}).catch(() => {});
			},
			removeDomainitem(index, qindex, dindex) {
				let dlist = this.list[index].qlist[qindex].option.deleteIndex(dindex);
				this.list[index].qlist[qindex].option = dlist;
			},
			itemSortdown(index, qindex, type) {
				var listItem = this.list[index].qlist[qindex];
				var sortList = this.list[index].qlist;
				var serial_number = listItem.serial_number;
				listItem.changeButton = false;
				if(type == 'up') {
					let uitem = this.list[index].qlist[qindex - 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.serial_number;
						uitem.serial_number = serial_number;
						listItem.serial_number = usort;
					} else {
						this.$message({
							type: 'error',
							message: '已经是第一题，无法继续上移!'
						});
					}
				} else if(type == 'down') {
					let uitem = this.list[index].qlist[qindex + 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.serial_number;
						uitem.serial_number = serial_number;
						listItem.serial_number = usort;
					} else {
						this.$message({
							type: 'error',
							message: '已经是最后一题，无法继续下移!'
						});
					}
				} else if(type == 'jumpitem') {
					let num = "";
					num = this.list[index].qlist[qindex].poSition;
					if(window.isNaN(num)) {
						this.$message({
							type: 'error',
							message: '输入有误，无法进行跳转!'
						});
						return;
					}
					let jumpnum = parseInt(num);
					if(jumpnum != serial_number) {
						let uitem = this.list[index].qlist[jumpnum - 1];
						if(uitem != undefined && uitem != null) {
							let usort = uitem.serial_number;
							uitem.serial_number = serial_number;
							uitem.poSition = '';
							listItem.serial_number = usort;
							//							listItem.changeButton = !listItem.changeButton;
							listItem.poSition = '';
						} else {
							this.$message({
								type: 'error',
								message: '找不到第' + jumpnum + '题，无法进行跳转!'
							});
						}
					} else {
						this.$message({
							type: 'error',
							message: '已经是第' + jumpnum + '题，无法进行跳转!'
						});
					}
				}
				sortList.sort(function(a, b) {
					return a.serial_number - b.serial_number;
				});

				this.list[index].qlist = sortList;

			},
			domainSortdown(index, qindex, dindex, type) {
				var sdomainItem = this.list[index].qlist[qindex].option[dindex];
				var sortList = this.list[index].qlist[qindex].option;
				var sortId = sdomainItem.order_num; //排序
				if(type == "up") {
					let uitem = this.list[index].qlist[qindex].option[dindex - 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.order_num;
						uitem.order_num = sortId;
						sdomainItem.order_num = usort;
					}
				} else {
					let uitem = this.list[index].qlist[qindex].option[dindex + 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.order_num;
						uitem.order_num = sortId;
						sdomainItem.order_num = usort;
					}
				}
				sortList.sort(function(a, b) {
					return a.order_num - b.order_num;
				});
				this.list[index].qlist[qindex].option = sortList;

			},
			openModel() {
				let self = this;
				var option = {};
				option.mod_name = "模块名称";
				option.qtitle = self.list.length + 1 + '、';
				option.id = 0;
				option.sortId = 0;
				option.pid = this.subId;
				option.sortId = 0;
				option.qlist = [];
				this.$post("/Home/Subject/createNewMod", {
					pid: this.subId,
					chief: this.parentModle || 0,
					mod_name: option.mod_name
				}).then((res) => {
					option.id = res.id;
					self.list.push(option);
				});
			},
			submitForm(item, index) {
				let subModel = JSON.parse(JSON.stringify(item));
				delete subModel.changeButton;
				delete subModel.edittextinput;
				delete subModel.show;
				delete subModel.poSition;
				//delete subModel.id; //如果 id=0,表示新加，有值表示修改
				let self = this; //fill single  multiple uploadimg loCation fractions 不需要特殊处理
				switch(item.sub_cat) {
					case "multistage":
						{
							let olist = subModel.olist; //多级下拉转换
							let svalue = parseInt(subModel.value); //选择显示几个
							delete subModel.olist;
							delete subModel.doptions;
							delete subModel.value;
							subModel.option = [{
								"default_choose": svalue,
								"option_name": olist
							}];
						}
						break;
					case "single":
						{
							delete subModel.default_choose;
						}
						break;
					case "comprehensive":
						{

						}
						break;
					case "multiple":
						{
							delete subModel.checkedGroup;
						}
						break;
				}
				console.log(subModel);
				this.$post("/Home/Subject/createNewItem", subModel).then((res) => {
					item.id = res.id;
				});
			},
			finishSub() {
				this.$confirm('您确定要完成问卷吗?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					let SubInfo = {
						id: this.subId,
						name: this.questiontitle,
						description: this.contentText,
						mod: []
					}
					for(var i = 0; i < this.list.length; i++) {
						let modoption = {};
						modoption.id = this.list[i].id;
						modoption.mod_name = this.list[i].mod_name;
						modoption.item = [];
						for(var j = 0; j < this.list[i].qlist.length; j++) {
							let jitem = {
								id: this.list[i].qlist[j].id,
								order: this.list[i].qlist[j].serial_number
							}
							modoption.item.push(jitem);
						}
						SubInfo.mod.push(modoption);
					}
					this.$post("/Home/Subject/finishSub", SubInfo).then((res) => {
						this.$alert('操作成功！', '提示', {
							confirmButtonText: '确定',
							callback: action => {
								this.$router.push({
									path: '/questionNaire'
								})
							}
						});
					});
				}).catch(() => {});

			}
		},
		mounted: function() {
			console.log(this.activeNames);
		},
		created() {
			this.subId = this.$route.query.questionId;
			
			this.$post("/Home/Subject/getSingleSub", {
				id: this.subId
			}).then((res) => {
				this.contentText = res.description || "";
				this.questiontitle = res.sub_name || "";
				//let modlist = res.mod;
				let aa = res.mod;
				let modlist = [];
				//modlist.push(aa[203]);
				//				modlist.push(aa[177]);
//				debugger
				for(var k = 0; k < modlist.length; k++) {
					var option = {};
//					debugger
					option.mod_name = modlist[k].mod_name;
					option.qtitle = this.list.length + 1 + '、';
					option.id = modlist[k].id;
					option.sortId = modlist[k].order_num;
					option.qlist = [];
					option.pid = modlist[k].pid;
					for(var n = 0; n < modlist[k].item.length; n++) {
						var sub_cat = modlist[k].item[n].sub_cat;
						switch(sub_cat) {
							case "fill":
								{
									let ifill = JSON.parse(JSON.stringify(ofill));
									for(var km in ifill) {
										modlist[k].item[n].hasOwnProperty(km) && (ifill[km] = modlist[k].item[n][km])
									}
									option.qlist.push(ifill);
								}
								break;
							case "single":
								{
									let isingle = JSON.parse(JSON.stringify(osingle));
									for(var km in isingle) {
										modlist[k].item[n].hasOwnProperty(km) && (isingle[km] = modlist[k].item[n][km])
									}
									option.qlist.push(isingle);
								}
								break;
							case "multiple":
								{
									let imultiple = JSON.parse(JSON.stringify(omultiple));
									for(var km in imultiple) {
										modlist[k].item[n].hasOwnProperty(km) && (imultiple[km] = modlist[k].item[n][km])
									}
									option.qlist.push(imultiple);
								}
								break;
							case "multistage":
								{
									let imultistage = JSON.parse(JSON.stringify(omultistage));
									for(var km in imultistage) {
										modlist[k].item[n].hasOwnProperty(km) && (imultistage[km] = modlist[k].item[n][km])
									}
									option.qlist.push(imultistage);
								}
								break;
							case "uploadimg":
								{
									let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
									for(var km in iuploadimg) {
										modlist[k].item[n].hasOwnProperty(km) && (iuploadimg[km] = modlist[k].item[n][km])
									}
									option.qlist.push(iuploadimg);
								}
								break;
							case "loCation":
								{
									let iloCation = JSON.parse(JSON.stringify(oloCation));
									for(var km in iloCation) {
										modlist[k].item[n].hasOwnProperty(km) && (iloCation[km] = modlist[k].item[n][km])
									}
									option.qlist.push(iloCation);
								}
								break;
							case "fractions":
								{
									let ifractions = JSON.parse(JSON.stringify(ofractions));
									for(var km in ifractions) {
										modlist[k].item[n].hasOwnProperty(km) && (ifractions[km] = modlist[k].item[n][km])
									}
									option.qlist.push(ifractions);
								}
								break;
							case "comprehensive":
								{

								}
								break;
							default:
								break;
						}
					}
					this.list.push(option);
				}
			});
		},
		components: {
			
			topic,
			fill,
			single,
			multiple,
			multistage,
			loCation,
			uploadimg,
			fractions,
			comprehensive
		}
	}
</script>
<style>
	.conBottom .el-collapse-item__wrap {
		border: none;
		border-radius: 0 0 4px 4px;
	}
	
	.conBottom .el-collapse-item__header {
		padding-left: 22px;
		border: none;
		border-radius: 4px;
	}
	
	.conBottom .el-collapse-item__content {
		padding: 10px 0;
		border-top: 1px solid #ebeef5;
	}
	
	.topicdropdown.el-popper[x-placement^=bottom] .popper__arrow::after {
		border-bottom-color: #005ad4;
	}
	
	.topicdropdown.el-popper[x-placement^=top] .popper__arrow::after {
		border-top-color: #005ad4;
	}
	
	.questiontitle .el-input__inner {
		border: none;
		text-align: center;
	}
	
	.questiontitle .el-textarea__inner {
		border: none;
		text-align: left;
	}
	
	.titlename .el-input__inner {
		width: 40%;
		margin-top: 5px;
		border: none;
	}
	
	.topic .el-form-item__label {
		display: block;
		width: 100%;
		text-align: left;
	}
</style>
<style scoped="scoped" lang="scss">
	* {
		box-sizing: border-box;
	}
	
	.edit_tempbg {
		background-color: #f3f3f3;
	}
	
	.top {
		padding: 29px 0;
		background-color: #fff;
		>.el-col {
			display: flex;
			justify-content: center;
			border-right: 2px solid #ccc;
			height: 10px;
			align-items: center;
			&:nth-of-type(1) {
				color: #0b61d6;
			}
			&:nth-of-type(2) {
				border-right: none;
			}
			>i {
				margin-right: 10px;
				font-size: 18px;
				font-weight: bold;
			}
		}
	}
	
	.editTemContain {
		padding: 68px 120px 0;
		background-color: #f3f3f3;
		height: 100%;
		>div {
			background-color: #fff;
		}
	}
	
	.conTop {
		padding: 15px 0;
		background: #303033;
		color: #fff;
		font-size: 14px;
		i {
			font-size: 20px;
			margin-right: 18px;
			display: inline-block;
			vertical-align: middle;
		}
	}
	
	.conBottom {
		padding: 60px;
		background-color: #fff;
		font-size: 14px;
	}
	
	.conBottomT {
		padding-bottom: 35px;
		border-bottom: 1px dashed #e4e4e4;
		margin-bottom: 35px;
		.el-input {
			.el-input__inner {
				border: none;
				text-align: center;
			}
		}
	}
	
	h6 {
		text-align: center;
		padding-bottom: 40px;
	}
	
	.el-collapse-item {
		border-color: #e0e0e0;
		border-radius: 4px;
		margin-bottom: 20px;
		border: 1px solid #ebeef5;
	}
	
	.el-collapse {
		border: none;
	}
	
	.new {
		background: url(../../statics/images/newquestion.png) no-repeat;
		position: absolute;
		top: 13px;
		display: inline-block;
		left: 17px;
		height: 20px;
		width: 20px;
	}
	
	.newitem {
		position: absolute;
		top: 0;
		right: 100px;
		background: #005ad4;
		color: #fff;
		z-index: 100;
		padding: 10px 0;
		li {
			&:nth-of-type(1) {
				padding: 7px 30px 7px 50px;
			}
			padding:7px 60px 7px 20px;
		}
	}
	
	.el-dropdown {
		position: absolute;
		top: 0;
		right: 100px;
		background: #005ad4;
		color: #fff;
		z-index: 100;
		padding: 15px 40px;
	}
	
	.el-dropdown-menu {
		/*top:170px !important;*/
		padding: 10px 20px;
		background: #005ad4;
	}
	
	.el-dropdown-menu__item {
		color: #fff;
	}
	
	.el-popper[x-placement^=bottom] .popper__arrow::after {
		border-bottom-color: #005ad4;
	}
	
	.edit_item {
		position: relative;
	}
	
	.questiontitle .el-input__inner {
		border: none;
		text-align: center;
	}
	
	.titlename {
		position: absolute;
		top: 0;
		left: 50px;
		z-index: 200;
		width: 50%;
	}
</style>