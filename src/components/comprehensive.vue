<template>
	<div class="compre">
		<el-form v-model="activeNames" @change="handleChange">
			<el-form-item class="edit_item" :label="item.qtitle+comtaccord">
				<el-input v-model="comitem.title" placeholder="综合题名称" class="titlename"></el-input>
				<div class="compretopright">
					<el-button @click.stop.prevent="relevance(item,index)" type="primary" plain class="molrelevance">关联逻辑</el-button>
					<relevance :qlist="qlist" :relatetype="relatetype" :list="list" :item="item" @canclerelevance='canclerelevance' :index="index" :qindex="qindex" @surerelevance="surerelevance"></relevance>
					<span @click.stop.prevent="deletecomp" class="deletecomp">删除</span>
					<span @click.stop.prevent="changeposition(comitem)" class="oposition">位置变更</span>
					<div class="changeposition" v-show="comitem.changeButton">
						<el-button type="info" plain @click.stop.prevent="itemSortdownc(item,index,qindex,'up')">上移一题</el-button>
						<el-button type="info" plain @click.stop.prevent="itemSortdownc(item,index,qindex,'down')">下移一题</el-button>
						<div>移至【
							<el-input v-model="comitem.poSition" class="inputposition"></el-input>】题
							<el-button type="primary" plain class="positionsure" @click.stop.prevent="itemSortdownc(item,index,qindex,'jumpitem')">确定</el-button>
						</div>
					</div>
				</div>
				<el-dropdown placement="bottom">
					<span class="el-dropdown-link">创建小题</span>
					<el-dropdown-menu slot="dropdown" class="compretopicdropdown" placement="bottom">
						<el-dropdown-item @click.native="addItem(index,'fill')">填空题</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'single')">选择题</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'multiple')">多选题</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'loCation')">位置上传</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'uploadimg')">图片上传</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'multistage')">多级下拉</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'fractions')">分数题</el-dropdown-item>
						<el-dropdown-item @click.native="addItem(index,'signature')">电子签名</el-dropdown-item>
					</el-dropdown-menu>
				</el-dropdown>

				<div class="topic" v-for="(qitem,qindex) in comitem.qlist" :key="qindex">
					<template v-if="qitem.sub_cat=='fill'">
						<fill :item="qitem" @removeDomain="removeDomain" :taccord="taccord" :list="list" :qlist="comitem.qlist" :type="type" :index="index" :qindex="qindex" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status" :relatetype="relatetype"></fill>
					</template>
					<template v-if="qitem.sub_cat=='single'">
						<single :item="qitem" @changeDomainRadio="changeDomainRadio" :list="list" :relatetype="relatetype" :type="type" :qlist="comitem.qlist" :taccord="taccord" @addDomain="addDomain" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></single>
					</template>
					<template v-if="qitem.sub_cat=='multiple'">
						<multiple :item="qitem" @addDomain="addDomain" :list="list" :taccord="taccord" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></multiple>
					</template>
					<template v-if="qitem.sub_cat=='multistage'">
						<multistage :item="qitem" :taccord="taccord" :list="list" :index="index" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></multistage>
					</template>
					<template v-if="qitem.sub_cat=='loCation'">
						<loCation :item="qitem" :taccord="taccord" :list="list" :index="index" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></loCation>
					</template>
					<template v-if="qitem.sub_cat=='uploadimg'">
						<uploadimg :item="qitem" :taccord="taccord" :list="list" :index="index" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></uploadimg>
					</template>
					<template v-if="qitem.sub_cat=='fractions'">
						<fractions :item="qitem" :taccord="taccord" :list="list" :index="index" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></fractions>
					</template>
					<template v-if="qitem.sub_cat=='signature'">
						<signature :item="qitem" :taccord="taccord" :list="list" :index="index" :type="type" :relatetype="relatetype" :qlist="comitem.qlist" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></signature>
					</template>
				</div>
			</el-form-item>

		</el-form>
	</div>
</template>

<script>
	import headTop from 'view/head/headTop.vue';
	import topic from 'components/topic.vue';
	import fill from 'components/fill.vue';
	import single from 'components/single.vue';
	import multiple from 'components/multiple.vue';
	import multistage from 'components/multistage.vue';
	import uploadimg from 'components/uploadimg.vue';
	import loCation from 'components/loCation.vue';
	import fractions from 'components/fractions.vue';
	import signature from 'components/signature.vue';
	import { Message } from "element-ui";
	import relevance from './relevance.vue';
	import { ofill, osingle, omultiple, omultistage, ouploadimg, oloCation, ofractions, osignature } from "./itemType";
	export default {
		data() {
			return {
				contentText: '',
				questiontitle: '',
				activeNames: [],
				region: "",
				modelId: "",
				taccord: ") ",
				relevanceshow: false
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
			type: {
				type: String,
				default: ''
			},
			relatetype: {
				type: String,
				default: ""
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
			comitem: {
				type: Object,
				default: {}
			},
			comtaccord: {
				type: String,
				default: ""
			},
			status: {
				type: String,
				default: "1"
			},

		},
		created() {
			//			debugger
			//let aa = this.comitem;

		},
		methods: {
			handleChange(val) {
				//				console.log(val);
			},
			addfill(index) {
				ofill.show = true;
				ofill.edittextinput = true;
				//let ix = this.comitem.qlist.length + 1;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let ifill = JSON.parse(JSON.stringify(ofill));
				ifill.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;;
				ifill.edittextinput = true;
				ifill.pid = this.comitem.id;
				ifill.serial_number = ix;
				ifill.qtitle = ix;
				this.comitem.qlist.push(ifill);
				this.resetComOrder(this.comitem.qlist);
				return ifill;
			},
			addsingle(index) {
				osingle.show = true;
				osingle.edittextinput = true;
				//				let ix = this.comitem.qlist.length + 1;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let isingle = JSON.parse(JSON.stringify(osingle));
				isingle.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;;

				isingle.pid = this.comitem.id;
				isingle.serial_number = ix;
				isingle.qtitle = ix;
				isingle.edittextinput = true;
				this.comitem.qlist.push(isingle);
				this.resetComOrder(this.comitem.qlist);
				return isingle;
			},
			addmultiple(index) {
				omultiple.show = true;
				omultiple.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let imultiple = JSON.parse(JSON.stringify(omultiple));
				imultiple.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;;
				imultiple.pid = this.comitem.id;
				imultiple.serial_number = ix;
				imultiple.qtitle = ix;
				imultiple.edittextinput = true;
				this.comitem.qlist.push(imultiple);
				this.resetComOrder(this.comitem.qlist);
				return imultiple;
			},
			addmultistage(index) {
				omultistage.show = true;
				omultistage.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let imultistage = JSON.parse(JSON.stringify(omultistage));
				imultistage.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;
				imultistage.pid = this.comitem.id;
				imultistage.serial_number = ix;
				imultistage.qtitle = ix;
				imultistage.edittextinput = true;
				this.comitem.qlist.push(imultistage);
				this.resetComOrder(this.comitem.qlist);
				return imultistage;
			},
			adduploadimg(index) {
				ouploadimg.show = true;
				ouploadimg.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
				iuploadimg.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;;
				iuploadimg.pid = this.comitem.id;
				iuploadimg.serial_number = ix;
				iuploadimg.qtitle = ix;
				iuploadimg.edittextinput = true;
				this.comitem.qlist.push(iuploadimg);
				this.resetComOrder(this.comitem.qlist);
				return iuploadimg
			},
			addloCation(index) {
				oloCation.show = true;
				oloCation.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let iloCation = JSON.parse(JSON.stringify(oloCation));
				iloCation.ppid = this.$route.query.questionId ? this.$route.query.questionId : this.$route.query.templateId;;
				iloCation.pid = this.comitem.id;
				iloCation.serial_number = ix;
				iloCation.qtitle = ix;
				iloCation.edittextinput = true;
				this.comitem.qlist.push(iloCation);
				this.resetComOrder(this.comitem.qlist);
				return iloCation;
			},
			addfractions(index) {
				ofractions.show = true;
				ofractions.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let ifractions = JSON.parse(JSON.stringify(ofractions));
				ifractions.ppid = this.$route.query.questionId;
				ifractions.pid = this.comitem.id;
				ifractions.serial_number = ix;
				ifractions.qtitle = ix;
				ifractions.edittextinput = true;
				this.comitem.qlist.push(ifractions);
				this.resetComOrder(this.comitem.qlist);
				return ifractions;
			},
			addsignature(index) {
				osignature.show = true;
				osignature.edittextinput = true;
				let ix = this.comitem.qlist.length == 0 ? 1 : Math.max.apply(Math, this.comitem.qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let isignature = JSON.parse(JSON.stringify(osignature));
				isignature.ppid = this.$route.query.questionId;
				isignature.pid = this.comitem.id;
				isignature.serial_number = ix;
				isignature.qtitle = ix;
				isignature.edittextinput = true;
				this.comitem.qlist.push(isignature);
				this.resetComOrder(this.comitem.qlist);
				return isignature;
			},
			relevance(item,index) {
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
				item.show = true;
				item.edittextinput = true;
			},
			canclerelevance(item) {
				item.relevanceshow = false;
				item.show = true;
				item.edittextinput = true;
			},
			resetComOrder(list) {
				var orderId = 1;
				for(var k = 0; k < list.length; k++) {
					list.sort(function(a, b) {
						return a.serial_number - b.serial_number;
					});
					list[k].qtitle = orderId;
					orderId++;
				}
			},
			addItem(index, type) {
				var that = this;
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				let item = new Promise((resolve, reject) => {
					let additem;
					switch(type) {
						case "fill":
							{
								additem = this.addfill(index);
							}
							break;
						case "single":
							{
								additem = this.addsingle(index);
							}
							break;
						case "multiple":
							{
								additem = this.addmultiple(index);
							}
							break;
						case "multistage":
							{
								additem = this.addmultistage(index);
							}
							break;
						case "uploadimg":
							{
								additem = this.adduploadimg(index);
							}
							break;
						case "loCation":
							{
								additem = this.addloCation(index);
							}
							break;
						case "fractions":
							{

								additem = this.addfractions(index);
							}
							break;
						case "signature":
							{
								additem = this.addsignature(index);
							}
							break;
						case "comprehensive":
							{
								additem = this.addcomprehensive(index);
							}
							break;
						default:
							break;
					}
					this.activeNames.indexOf(index) == -1 && this.activeNames.push(index);
					resolve(additem);
				})

				item.then(function(additem) {

					that.submitForm(additem, index);
				}).catch(function() {
					//failure
				})
			},
			addDomain(index, qindex) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				let sort = this.comitem.qlist[qindex].option.length + 1;
				let options = {
					id: 0,
					order_num: sort,
					option_name: "选项" + sort,
					default_choose: 0,
					related_sub: '',
					skip_sub: '',
					score: 0
				}
				this.comitem.qlist[qindex].option.push(options);
			},
			changeDomainRadio(index, qindex, v) {
				//				debugger
				if(v == this.comitem.qlist[qindex].default_choose) {
					this.comitem.qlist[qindex].default_choose = "";
				} else {
					this.comitem.qlist[qindex].default_choose = v;
				}
				let domainlist = this.comitem.qlist[qindex].option;
				for(let i in domainlist) {
					if(domainlist[i].option_name == v) {
						domainlist[i].default_choose = 1;
					} else {
						domainlist[i].default_choose = 0;
					}
				}
			},
			removeDomain(index, qindex) {
				this.$confirm('您确定要删除吗?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					let dmodel = this.comitem.qlist[qindex];

					if(dmodel.id != 0) {
						var deleteitem = ""
						if(this.$route.query.templateId) {
							deleteitem = "/Home/Tpl/deleteItem"
						} else {
							deleteitem = "/Home/Subject/deleteItem"
						}
						this.$post(deleteitem, {
							id: dmodel.id
						}).then((res) => {});
					}
					let nlist = this.comitem.qlist.deleteIndex(qindex);
					this.comitem.qlist = nlist;
					this.resetComOrder(this.comitem.qlist);
				}).catch(() => {});
			},
			deletecomp(item) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("deleCom", this.index, this.qindex);
				if(this.$route.query.templateId) {
					this.$post("/Home/Tpl/deleteMod", {
						"id": this.item.id
					})
				} else {
					this.$post("/Home/Subject/deleteMod", {
						"id": this.item.id
					})
				}

			},
			changeposition(item) {
				item.changeButton = !item.changeButton;
				
			},
			removeDomainitem(index, qindex, dindex) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				let dlist = this.comitem.qlist[qindex].domains.deleteIndex(dindex);
				this.comitem.qlist[qindex].domains = dlist;
			},
			itemSortdownc(item, index, qindex, type) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.$emit("itemSortdown", item, index, qindex, type);
			},
			itemSortdown: function(item, index, qindex, type) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				var listItem = this.comitem.qlist[qindex];
				var sortList = this.comitem.qlist;
				var serial_number = listItem.serial_number;
				listItem.changeButton = false;
				if(type == 'up') {

					let uitem = this.comitem.qlist[qindex - 1];
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
					let uitem = this.comitem.qlist[qindex + 1];
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
					num = this.comitem.qlist[qindex].poSition;
					if(window.isNaN(num)) {
						this.$message({
							type: 'error',
							message: '输入有误，无法进行跳转!'
						});
						return;
					}
					let jumpnum = parseInt(num);
					if(jumpnum != serial_number) {
						let uitem = this.comitem.qlist[jumpnum - 1];
						if(uitem != undefined && uitem != null) {
							let usort = uitem.serial_number;
							uitem.serial_number = serial_number;
							uitem.poSition = '';
							listItem.serial_number = usort;
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
//				sortList.sort(function(a, b) {
//					return a.serial_number - b.serial_number;
//				});
//				this.comitem.qlist = sortList;
this.resetComOrder(this.comitem.qlist)
				item.show = false;
				item.edittextinput = false;
			},
			submitForm(item, index) {
				let subModel = JSON.parse(JSON.stringify(item));
				item.show = false;
				item.edittextinput = false;
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
					case "multiple":
						{
							delete subModel.checkedGroup;
						}
						break;
				}
				if(this.$route.query.templateId) {
					this.$post("/Home/Tpl/createNewItem", subModel).then((res) => {
						item.id = res.id;
						if(res.option.length > 0) {
							for(var i = 0; i < res.option.length; i++) {
								item.option[i].id = res.option.filter(o => o.order_num == item.option[i].order_num)[0].id;
							}
						}

					});
				} else {
					this.$post("/Home/Subject/createNewItem", subModel).then((res) => {
						item.id = res.id;
						if(res.option.length > 0) {
							for(var i = 0; i < res.option.length; i++) {

								item.option[i].id = res.option.filter(o => o.order_num == item.option[i].order_num)[0].id;
							}
						}

					});
				}

			},
			domainSortdown(index, qindex, dindex, type) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				var sdomainItem = this.comitem.qlist[qindex].option[dindex];
				var sortList = this.comitem.qlist[qindex].option;
				var sortId = sdomainItem.order_num; //排序
				if(type == "up") {
					let uitem = this.comitem.qlist[qindex].option[dindex - 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.order_num;
						uitem.order_num = sortId;
						sdomainItem.order_num = usort;
					}
				} else {
					let uitem = this.comitem.qlist[qindex].option[dindex + 1];
					if(uitem != undefined && uitem != null) {
						let usort = uitem.order_num;
						uitem.order_num = sortId;
						sdomainItem.order_num = usort;
					}
				}
				sortList.sort(function(a, b) {
					return a.order_num - b.order_num;
				});
				this.comitem.qlist[qindex].option = sortList;
			}
		},
		mounted: function() {
			//			console.log(this.activeNames);
		},
		components: {
			headTop,
			topic,
			fill,
			single,
			multiple,
			multistage,
			loCation,
			uploadimg,
			fractions,
			signature,
			relevance
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
		width: 100%;
		margin-top: 5px;
		border: none;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	.topic .el-form-item__label {
		display: block;
		width: 100%;
		text-align: left;
	}
	
	.topic .compre .el-form-item__label {
		/*width: 0;
    text-align: right;
    margin-left: 50px;
        margin-top: 5px;*/
	}
	
	.topic .compre {
		padding-top: 10px;
	}
	
	.topic .compre>.el-form>.el-form-item {
		margin-bottom: 0;
	}
	
	.topic .compre>.el-form>.edit_item>.el-form-item__label {
		width: 0;
		text-align: right;
		margin-left: 43px;
		margin-top: 5px;
		float: left;
	}
	
	.topic .compre>.el-form>.edit_item>.el-form-item__content>.compretopright>.changeposition {
		right: 12%;
		top: 35px;
	}
	
	.topic .compre .titlename {
		position: initial;
		width: 45%;
	}
	/*.compre .el-dropdown{
		position: inherit !important;
	}*/
	
	.compre .el-dropdown {
		top: 6px !important;
	}
	
	.el-dropdown-menu:nth-of-type(2).el-popper[x-placement^=bottom] .popper__arrow::after {
		border-bottom-color: #fff;
	}
	
	.el-dropdown-menu:nth-of-type(2).el-popper[x-placement^=top] .popper__arrow::after {
		border-top-color: #fff;
	}
	
	.compre .changeposition .el-button {
		width: 100%;
		display: block;
	}
	
	.compre .changeposition .el-button.positionsure {
		width: 40px;
		display: inline-block;
		margin: 0;
		padding: 3px 0;
	}
	
	.compre .changeposition {}
	
	.el-collapse-item__wrap {
		overflow: initial;
	}
	
	.compre .changeposition .inputposition .el-input__inner {
		background-color: rgb(245, 245, 245);
		height: 14px;
		line-height: 14px;
		margin: 0;
		border: none;
		padding: 0 5px;
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
		/*background: url(../statics/images/newquestion.png) no-repeat;*/
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
		/*background: #005ad4;*/
		/*color: #fff;*/
		/*z-index: 100;
		padding: 10px 0;
		li {
			&:nth-of-type(1) {
				padding: 7px 30px 7px 50px;
			}
			padding:7px 60px 7px 20px;
		}*/
	}
	
	.el-dropdown {
		position: absolute;
		top: 12px;
		right: 65px;
		/*background: #005ad4;*/
		color: #299bfc;
		cursor: pointer;
		/*z-index: 100;
		padding: 15px 40px;*/
	}
	
	.topicdropdown.el-dropdown-menu {
		top: 170px !important;
		/*padding: 10px 20px;
		background: #005ad4;*/
	}
	/*.el-dropdown-menu__item {
		color: #fff;
	}*/
	
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
	
	.deletecomp {
		position: absolute;
		right: 40px;
		color: #299bfc;
		top: 12px;
	}
	/*.compreitem .el-dropdown{
		background:none;
		color:#299bfc;
	}*/
	
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
	
	.changeposition .inputposition .el-input__inner {
		background-color: rgb(245, 245, 245);
		height: 14px;
		line-height: 14px;
		margin: 0;
		border: none;
		padding: 0 5px;
	}
	
	.changeposition>div>span {
		display: inline-block;
		width: 0;
	}
	
	.compre .compretopright {
		width: 33%;
		display: inline-block;
		span {
			width: 30%;
			display: inline-block;
			text-align: center;
			cursor: pointer;
			&:hover {
				color: #005ad4;
			}
		}
	}
	
	.compre .deletecomp {
		position: initial;
	}
	
	.oposition {
		color: #299bfc;
	}
	
	.compre {
		.changeposition .inputposition .el-input__inner {
			background-color: rgb(245, 245, 245);
			height: 14px;
			line-height: 14px;
			margin: 0;
			border: none;
			padding: 0 5px;
		}
		.edit_item /deep/ .el-form-item__label {
			padding: 0 25px 0 0;
		}
	}
	
	.edit_item /deep/ .comrelevance {
		background: none;
		border: none;
		display: inline-block;
		padding: 0;
		width: 28%;
		&:hover {
			background: none;
			
			border-color:none;
			span{
				color: #005ad4;
			}
		}
	}
		.edit_item /deep/ .molrelevance.el-button--primary.is-plain{
		background: none;
		border: none;
		display: inline-block;
		padding: 0;
		width: 28%;
		&:hover {
			background: none;
			
			border-color:none;
			span{
				color: #005ad4;
			}
		}
		&:active{
			span{
				color:#3a8ee6;
			}
		}
	}
</style>