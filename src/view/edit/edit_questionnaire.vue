<template>
	<div class="edit_tempbg">
		<el-row :gutter="20" class="top">
			<!--<i class="el-icon-search"></i>预览</el-col>-->
			<el-col :offset="10" :span="3" @click.native="finishSub()"><i class="el-icon-check"></i>完成</el-col>
		</el-row>
		<div class="editTemContain">
			<div class="outconTop">
				<el-row type="flex" justify="end" class="conTop">

					<el-col :span="3" @click.native="openModel" class="openModel"><i class="el-icon-plus"></i>新建模块</el-col>

				</el-row>
			</div>
			<div class="conBottom">
				<div class="conBottomT">
					<el-input type="text" v-model="questiontitle" placeholder="问卷标题" class="questiontitle"></el-input>
					<el-input type="textarea" v-model="contentText" placeholder="问卷说明" class="questiontitle"></el-input>
				</div>

				<el-collapse v-model="activeNames" @change="handleChange">
					<div class="edit_item" v-for="(item,index) in list" :key="index">
						<template v-if="index!=0">
							<el-button @click="relevance(item,index)" type="primary" plain class="molrelevance">关联逻辑</el-button>
							<relevance :relevanceshow='relevanceshow' :qlist="qlist" :relatetype="'mol'" :list="list" :item="item" @canclerelevance='canclerelevance' :index="index" :qindex="qindex" @surerelevance="surerelevance"></relevance>	
						</template>
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
								<el-dropdown-item @click.native="addItem(index,'signature')">电子签名</el-dropdown-item>
								<el-dropdown-item @click.native="addItem(index,'comprehensive')">综合题</el-dropdown-item>
							</el-dropdown-menu>
						</el-dropdown>
						<el-input v-model="item.mod_name" placeholder="模块名称" class="titlename"></el-input>
						<el-collapse-item :title="item.qtitle" :name="index">

							<div class="topic" v-for="(qitem,qindex) in item.qlist" :key="qindex">

								<template v-if="qitem.sub_cat=='fill'">
									<fill :item="qitem" :list="list" :relatetype="relatetype" :qlist="item.qlist" @removeDomain="removeDomain" :taccord="taccord" :index="index" :qindex="qindex" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></fill>
								</template>
								<template v-if="qitem.sub_cat=='single'">
									<single :item="qitem" :list="list" :relatetype="relatetype" :qlist="item.qlist" :taccord="taccord" @changeDomainRadio="changeDomainRadio" @addDomain="addDomain" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></single>
								</template>
								<template v-if="qitem.sub_cat=='multiple'">
									<multiple :item="qitem" :list="list" @addDomain="addDomain" :relatetype="relatetype" :qlist="item.qlist"  :taccord="taccord" :index="index" :qindex="qindex" @removeDomainitem="removeDomainitem" @removeDomain="removeDomain" @domainSortdown="domainSortdown" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></multiple>
								</template>
								<template v-if="qitem.sub_cat=='multistage'">
									<multistage :item="qitem" :list="list" :taccord="taccord" :relatetype="relatetype" :qlist="item.qlist"  :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></multistage>
								</template>
								<template v-if="qitem.sub_cat=='loCation'">
									<loCation :item="qitem" :list="list" :taccord="taccord" :relatetype="relatetype" :qlist="item.qlist"  :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></loCation>
								</template>
								<template v-if="qitem.sub_cat=='uploadimg'">
									<uploadimg :item="qitem" :list="list" :taccord="taccord"  :relatetype="relatetype" :qlist="item.qlist"  :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></uploadimg>
								</template>
								<template v-if="qitem.sub_cat=='fractions'">
									<fractions :item="qitem" :list="list" :taccord="taccord" :relatetype="relatetype" :qlist="item.qlist"  :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></fractions>
								</template>
								<template v-if="qitem.sub_cat=='signature'">
									<signature :item="qitem" :list="list" :taccord="taccord"  :relatetype="relatetype" :qlist="item.qlist" :index="index" :qindex="qindex" @removeDomain="removeDomain" @itemSortdown="itemSortdown" @submitForm="submitForm" :status="status"></signature>
								</template>

								<template v-if="qitem.sub_cat=='comprehensive'">
									<comprehensive :list="list" :relatetype="'molcom'" :item="qitem" :qlist="item.qlist" @deleCom="delem"  :comtaccord="comtaccord" :index="index" :comitem="qitem" :status="status" :qindex="qindex" @itemSortdown="itemSortdown" @changeDomainRadio="changeDomainRadio"></comprehensive>
								</template>
							</div>
						</el-collapse-item>
						<div class="quetiondelete" @click.stop="removeMode(item)"><i class="el-icon-delete"></i></div>

					</div>
				</el-collapse>

			</div>

		</div>

	</div>
</template>

<script>
	import { jsNumDX } from 'javascripts/utils/index';
	import topic from 'components/topic.vue';
	import fill from 'components/fill.vue';
	import single from 'components/single.vue';
	import multiple from 'components/multiple.vue';
	import multistage from 'components/multistage.vue';
	import uploadimg from 'components/uploadimg.vue';
	import loCation from 'components/loCation.vue';
	import signature from 'components/signature.vue';
	import fractions from 'components/fractions.vue';
	import { Message } from "element-ui";
	import comprehensive from 'components/comprehensive.vue';
	import { ofill, osingle, omultiple, omultistage, ouploadimg, oloCation, ofractions, ocomprehensive, osignature } from "components/itemType";
	import relevance from 'components/relevance.vue';
	export default {
		data() {
			return {
				contentText: '',
				questiontitle: '',
				activeNames: [],
				list: [],
				qlist:[],
				qindex:0,
				taccord: "、",
				comtaccord: "、",
				subId: "",
				parentModle: 0,
				serial_number: 0,
				status: "",
				SubInfo: {},
				relatetype: "moloption", //mol 模块关联  moloption 模块题目关联 molcom综合题关联
				relevanceshow: false
			}
		},
		methods: {
			handleChange(val) {
				console.log(val);
			},
			addfill(index) {
				//let ix = this.list[index].qlist.length + 1;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				ofill.show = true;
				ofill.edittextinput = true;
				let ifill = JSON.parse(JSON.stringify(ofill));
				ifill.ppid = this.subId;
				ifill.pid = this.list[index].id;
				ifill.serial_number = ix;
				ifill.qtitle = ix;
				this.list[index].qlist.push(ifill);
			},
			addsingle(index) {
				osingle.show = true;
				osingle.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let isingle = JSON.parse(JSON.stringify(osingle));
				isingle.ppid = this.subId;
				isingle.pid = this.list[index].id;
				isingle.serial_number = ix;
				isingle.qtitle = ix;
				this.list[index].qlist.push(isingle);
			},
			addmultiple(index) {
				omultiple.show = true;
				omultiple.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let imultiple = JSON.parse(JSON.stringify(omultiple));
				imultiple.ppid = this.subId;
				imultiple.pid = this.list[index].id;
				imultiple.serial_number = ix;
				imultiple.qtitle = ix;
				this.list[index].qlist.push(imultiple);

			},
			relevance(item,index) {
				if(this.status != "1" && this.type != '0' && !this.$route.query.templateId) {

					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.relevanceshow = true;
								
			},
			surerelevance() {
				this.relevanceshow = false;
			},
			canclerelevance(item) {
				this.relevanceshow = false;
			},
			addmultistage(index) {
				omultistage.show = true;
				omultistage.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let imultistage = JSON.parse(JSON.stringify(omultistage));
				imultistage.ppid = this.subId;
				imultistage.pid = this.list[index].id;
				imultistage.serial_number = ix;
				imultistage.qtitle = ix;
				this.list[index].qlist.push(imultistage);
			},
			adduploadimg(index) {
				ouploadimg.show = true;
				ouploadimg.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
				iuploadimg.ppid = this.subId;
				iuploadimg.pid = this.list[index].id;
				iuploadimg.serial_number = ix;
				iuploadimg.qtitle = ix;
				this.list[index].qlist.push(iuploadimg);
			},
			addloCation(index) {
				oloCation.show = true;
				oloCation.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let iloCation = JSON.parse(JSON.stringify(oloCation));
				iloCation.ppid = this.subId;
				iloCation.pid = this.list[index].id;
				iloCation.serial_number = ix;
				iloCation.qtitle = ix;
				this.list[index].qlist.push(iloCation);
			},
			addsignature(index) {
				osignature.show = true;
				osignature.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let isignature = JSON.parse(JSON.stringify(osignature));
				isignature.ppid = this.subId;
				isignature.pid = this.list[index].id;
				isignature.serial_number = ix;
				isignature.qtitle = ix;
				this.list[index].qlist.push(isignature);
			},
			addfractions(index) {
				ofractions.show = true;
				ofractions.edittextinput = true;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				let ifractions = JSON.parse(JSON.stringify(ofractions));
				ifractions.ppid = this.subId;
				ifractions.pid = this.list[index].id;
				ifractions.serial_number = ix;
				ifractions.qtitle = ix;
				this.list[index].qlist.push(ifractions);

			},
			addcomprehensive(index) {
				//let ix = this.list[index].qlist.length + 1;
				//				let ix = Math.max.apply(Math, this.list[index].qlist.map(function(item) {
				//					return item.serial_number
				//				})) + 1;
				let ix = this.list[index].qlist.length == 0 ? 1 : Math.max.apply(Math, this.list[index].qlist.map(function(item) {
					return item.serial_number
				})) + 1;
				var option = {};
				option.title = "综合题名称";
				option.qtitle = ix;
				option.qlist = [];
				option.changeButton = false;
				option.serial_number = ix;
				option.id = 0;
				option.ppid = this.subId;
				option.pid = this.list[index].id;
				option.sub_cat = "comprehensive";
				option.poSition = "";
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
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				this.list[index].qlist.splice(pindex, 1);

			},
			addItem(index, type) {
				let that = this;
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}

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
					case "signature":
						{
							this.addsignature(index);
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
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				let sort = this.list[index].qlist[qindex].option.length + 1;
				let options = {
					id: 0,
					order_num: sort,
					option_name: "选项" + sort,
					score: 0,
					default_choose: 0,
					related_sub: '',
					skip_sub: ''
				}
				this.list[index].qlist[qindex].option.push(options);

			},
			changeDomainRadio(index, qindex, v) {
				if(v == this.list[index].qlist[qindex].default_choose) {
					this.list[index].qlist[qindex].default_choose = "";
				} else {
					this.list[index].qlist[qindex].default_choose = v;
				}

				let domainlist = this.list[index].qlist[qindex].option;
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
					let dmodel = this.list[index].qlist[qindex];
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
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				debugger
				let dlist = this.list[index].qlist[qindex].option.deleteIndex(dindex);
				this.list[index].qlist[qindex].option = dlist;
			},
			itemSortdown(item, index, qindex, type) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
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
				item.show = false;
				item.edittextinput = false;

			},
			domainSortdown(index, qindex, dindex, type) {
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
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
				if(this.status != "1") {
					this.$message({
						type: 'error',
						message: '当前问卷状态无法进行此操作'
					});
					return;
				}
				let self = this;
				var option = {};
				var opcount = self.list.length == 0 ? 0 : Math.max.apply(Math, self.list.map(function(item) {
					return item.serial_number
				}));
				opcount = opcount + 1;
				option.mod_name = "模块名称";
				option.qtitle = jsNumDX(opcount);
				option.id = 0;
				option.serial_number = opcount;
				option.pid = this.subId;
				option.qlist = [];
				this.$post("/Home/Subject/createNewMod", {
					pid: this.subId,
					chief: 0,
					mod_name: option.mod_name
				}).then((res) => {
					option.id = res.id;
					self.list.push(option);
				});
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
					case "comprehensive":
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
						modoption.serial_number = this.list[i].serial_number;
						modoption.item = [];
						for(var j = 0; j < this.list[i].qlist.length; j++) {
							if(this.list[i].qlist[j].sub_cat == "comprehensive") {
								let bmodoption = {};
								bmodoption.id = this.list[i].qlist[j].id;
								bmodoption.mod_name = this.list[i].qlist[j].title;
								bmodoption.serial_number = this.list[i].qlist[j].serial_number;
								bmodoption.item = [];
								for(var b = 0; b < this.list[i].qlist[j].qlist.length; b++) {
									var jitem = {};
									jitem.id = this.list[i].qlist[j].qlist[b].id;
									jitem.order = this.list[i].qlist[j].qlist[b].serial_number;
									bmodoption.item.push(jitem);
								}
								SubInfo.mod.push(bmodoption);
							} else {
								var jitem = {};
								jitem.id = this.list[i].qlist[j].id;
								jitem.order = this.list[i].qlist[j].serial_number;
								modoption.item.push(jitem);
								if(SubInfo.mod.filter(o => o.id == modoption.id).length == 0) {
									SubInfo.mod.push(modoption);
								}
							}
						}
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

			},
			casbreakoptions(sub_cat, fatheritem, ly) {

				switch(sub_cat) {
					case "fill":
						{
							let ifill = JSON.parse(JSON.stringify(ofill));
							for(var km in ifill) {
								fatheritem.hasOwnProperty(km) && (ifill[km] = fatheritem[km])
							}
							ly.qlist.push(ifill);
						}
						break;
					case "single":
						{
							let isingle = JSON.parse(JSON.stringify(osingle));
							for(var km in isingle) {
								fatheritem.hasOwnProperty(km) && (isingle[km] = fatheritem[km])
							}
							let ly1 = fatheritem.option.filter(o => o.default_choose == 1);
							(isingle.default_choose == "" && ly1.length > 0) && (isingle.default_choose = ly1[0].option_name);

							ly.qlist.push(isingle);
						}
						break;
					case "multiple":
						{
							let imultiple = JSON.parse(JSON.stringify(omultiple));
							for(var km in imultiple) {
								fatheritem.hasOwnProperty(km) && (imultiple[km] = fatheritem[km])
							}
							ly.qlist.push(imultiple);
						}
						break;
					case "multistage":
						{
							let imultistage = JSON.parse(JSON.stringify(omultistage));
							for(var km in imultistage) {
								fatheritem.hasOwnProperty(km) && (imultistage[km] = fatheritem[km])
							}
							ly.qlist.push(imultistage);
						}
						break;
					case "uploadimg":
						{
							let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
							for(var km in iuploadimg) {
								fatheritem.hasOwnProperty(km) && (iuploadimg[km] = fatheritem[km])
							}
							ly.qlist.push(iuploadimg);
						}
						break;
					case "loCation":
						{
							let iloCation = JSON.parse(JSON.stringify(oloCation));
							for(var km in iloCation) {
								fatheritem.hasOwnProperty(km) && (iloCation[km] = fatheritem[km])
							}
							ly.qlist.push(iloCation);
						}
						break;
					case "fractions":
						{
							let ifractions = JSON.parse(JSON.stringify(ofractions));
							for(var km in ifractions) {
								fatheritem.hasOwnProperty(km) && (ifractions[km] = fatheritem[km])
							}

							ly.qlist.push(ifractions);
						}
						break;
					case "comprehensive":
						{

						}
						break;
					default:
						break;
				}

			},
			removeMode(item) {
				this.$confirm('您确定要删除该模块，是否继续？', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					this.$post("/Home/Subject/deleteMod", {
						"id": item.id
					}).then((res) => {
						this.$alert('删除成功！', '提示', {
							confirmButtonText: '确定',
							callback: action => {
								this.getListArrary(this.subId);

							}
						});
					});
				}).catch(() => {
					this.$message({
						type: 'info',
						message: '已取消删除'
					});
				});
			},
			getItemOptions(itemList) {
				let resList = [];
				Array.from(itemList).forEach((obj, index) => {
					let sub_cat = obj.sub_cat;
					let fatheritem = obj;
					switch(sub_cat) {
						case "fill":
							{
								let ifill = JSON.parse(JSON.stringify(ofill));
								for(var km in ifill) {
									fatheritem.hasOwnProperty(km) && (ifill[km] = fatheritem[km])
								}
								resList.push(ifill);
							}
							break;
						case "single":
							{
								let isingle = JSON.parse(JSON.stringify(osingle));
								for(var km in isingle) {
									fatheritem.hasOwnProperty(km) && (isingle[km] = fatheritem[km])
								}
								let ly1 = fatheritem.option.filter(o => o.default_choose == 1);
								(isingle.default_choose == "" && ly1.length > 0) && (isingle.default_choose = ly1[0].option_name);
								resList.push(isingle);
							}
							break;
						case "multiple":
							{
								let imultiple = JSON.parse(JSON.stringify(omultiple));
								for(var km in imultiple) {
									fatheritem.hasOwnProperty(km) && (imultiple[km] = fatheritem[km])
								}
								resList.push(imultiple);
							}
							break;
						case "multistage":
							{
								let imultistage = JSON.parse(JSON.stringify(omultistage));
								for(var km in imultistage) {
									fatheritem.hasOwnProperty(km) && (imultistage[km] = fatheritem[km])
								}
								//还原初始化
								let iobj = fatheritem.option[0] || {};
								iobj.hasOwnProperty("default_choose") && (imultistage.value = iobj.default_choose);
								iobj.hasOwnProperty("option_name") && (imultistage.olist = iobj.option_name);

								resList.push(imultistage);
							}
							break;
						case "uploadimg":
							{
								let iuploadimg = JSON.parse(JSON.stringify(ouploadimg));
								for(var km in iuploadimg) {
									fatheritem.hasOwnProperty(km) && (iuploadimg[km] = fatheritem[km])
								}
								resList.push(iuploadimg);
							}
							break;
						case "loCation":
							{
								let iloCation = JSON.parse(JSON.stringify(oloCation));
								for(var km in iloCation) {
									fatheritem.hasOwnProperty(km) && (iloCation[km] = fatheritem[km])
								}
								resList.push(iloCation);
							}
							break;
						case "fractions":
							{
								let ifractions = JSON.parse(JSON.stringify(ofractions));
								for(var km in ifractions) {
									fatheritem.hasOwnProperty(km) && (ifractions[km] = fatheritem[km])
								}
								resList.push(ifractions);
							}
							break;
						case "signature":
							{
								let isignature = JSON.parse(JSON.stringify(osignature));
								for(var km in isignature) {
									fatheritem.hasOwnProperty(km) && (isignature[km] = fatheritem[km])
								}
								resList.push(isignature);
							}
							break;

					}
				});

				return resList;
			},
			getListArrary(subId) {
				this.list = [];
				this.$post("/Home/Subject/getSingleSub", {
					id: subId
				}).then((res) => {
					this.status = res.status + "";
					this.contentText = res.description || "";
					this.questiontitle = res.sub_name || "";
					let modlist = res.mod;
					for(var k = 0; k < modlist.length; k++) {
						var option = {};
						option.mod_name = modlist[k].mod_name;
						option.id = modlist[k].id;
						var serial_number = modlist[k].serial_number;
						try {
							serial_number = parseInt(serial_number);
						} catch(xm) {
							serial_number = 1;
						}
						option.serial_number = serial_number;
						option.qtitle = jsNumDX(serial_number);
						option.qlist = [];
						option.pid = modlist[k].pid;
						if(modlist[k].item.length != 0) {
							let opList = this.getItemOptions(modlist[k].item);
							option.qlist = opList;
						}
						if(modlist[k].mod && modlist[k].mod.length != 0) {
							for(var j = 0; j < modlist[k].mod.length; j++) {
								let icomprehensive = JSON.parse(JSON.stringify(ocomprehensive));
								for(var km in icomprehensive) {
									modlist[k].mod[j].hasOwnProperty(km) && (icomprehensive[km] = modlist[k].mod[j][km])
								}
								let copList = this.getItemOptions(modlist[k].mod[j].item);
								copList.sort(function(a, b) {
									return a.serial_number - b.serial_number;
								});
								icomprehensive.title = modlist[k].mod[j].mod_name;
								icomprehensive.qlist = copList;
								option.qlist.push(icomprehensive);
							}
						}
						option.qlist.sort(function(a, b) {
							return a.serial_number - b.serial_number;
						});
						this.list.push(option);
					}
				});
			}
		},
		mounted: function() {},
		created() {
			this.subId = this.$route.query.questionId;
			this.getListArrary(this.subId);
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
			comprehensive,
			signature,
			relevance
		}
	}
</script>
<style>
	.conBottom .el-collapse-item__wrap {
		border: none;
		overflow: initial;
		border-radius: 0 0 4px 4px;
	}
	
	.conBottom .el-collapse-item__header {
		padding-left: 22px;
		border: none;
		border-radius: 4px;
		font-weight: bold;
		font-size: 16px;
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
	
	.titlename .el-input__inner {
		width: 100%;
		margin-top: 5px;
		border: none;
		font-size: 14px;
		font-weight: bold;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	.edit_item>.titlename .el-input__inner {
		font-size: 16px;
		font-weight: bold;
	}
	
	.topic .el-form-item__label {
		display: block;
		width: 100%;
		text-align: left;
		font-size: 14px;
		font-weight: bold;
	}
	
	.el-form-item__content>.topic .el-form-item__label {
		font-weight: normal;
	}
	
	.questiontitle .el-input__inner {
		border: none;
		text-align: center;
		font-size: 20px;
		font-weight: bold;
	}
	
	.topic .el-form-item__content>.itemmust {
		top: -38px;
	}
	
	.el-radio:focus:not(.is-focus):not(:active):not(.is-disabled) .el-radio__inner {
		box-shadow: none;
	}
	
	.topic .avatar-uploader-icon {
		line-height: 100px !important;
	}
</style>
<style scoped="scoped" lang="scss">
	* {
		box-sizing: border-box;
	}
	
	.edit_tempbg {
		background-color: #f3f3f3;
		min-width: 1000px;
		height: 100%;
		overflow-x: hidden;
		padding-top: 30px;
		position: relative;
	}
	
	.top {
		margin-left: 0 !important;
		margin-right: 0 !important;
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		background: #fff;
		height: 50px;
		line-height: 50px;
		z-index: 20;
		>.el-col {
			display: flex;
			justify-content: center;
			height: 40px;
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
		width: 1000px;
		height: 100%;
		margin: 0 auto;
		background: #fff;
		padding-top: 90px;
		overflow-x: hidden;
		>div {
			background-color: #fff;
		}
	}
	
	.conTop {
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
		z-index: 3;
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
		right: 150px;
		background: #005ad4;
		color: #fff;
		z-index: 3;
		padding: 15px 40px;
		cursor: pointer;
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
	
	.titlename {
		position: absolute;
		top: 0;
		left: 50px;
		z-index: 3;
		width: 45%;
	}
	
	.quetiondelete {
		position: absolute;
		top: 10px;
		right: 60px;
		z-index: 3;
		height: 40px;
		width: 40px;
		cursor: pointer;
	}
	
	.quetiondelete i.el-icon-delete {
		font-size: 24px;
		color: #299BFC;
		&:hover {
			color: #005ad4;
		}
	}
	
	.openModel {
		/*position:absolute;*/
	}
	
	.outconTop {
		position: absolute;
		top: 45px;
		left: 0;
		width: 100%;
		z-index: 20;
		.conTop {
			background: #000;
			color: #fff;
			text-align: right;
			padding: 10px 20px;
			width: 1000px;
			margin: 0 auto;
		}
	}
	.molrelevance{
		background:none;
		border:none;
		padding:0;
		    position: absolute;
    top: 20px;
    right: 346px;
		&:hover{
			background:none;
			color:#005ad4;
		}
	}
</style>