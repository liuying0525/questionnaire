<template>
	<div v-show="relevanceshow" class="relevance">
		<div class="relevanceitem">
			<p>关联逻辑设置：</p>
			<ul class="relevanceitemcontent">
				<li>
					<span>当前题目：</span>
					<span>{{item.serial_number}}、{{item.title}}</span>
				</li>
				<li>
					<span>关联题目：</span>
					<span>	
						<el-select v-model="sItem" placeholder="请选择" @change="itemChange">
							<el-option v-for="(opitem,oindex) in slist" :key="oindex"  :label="opitem.name"  :value="opitem.id"></el-option>
						</el-select>
					</span>
				</li>
				<li>
					<span>题目选项：</span>
					<span>					
						<el-checkbox-group v-model="sOption" @change="ckchange">
						  <el-checkbox v-for="(checkoption,index) in coptions" :disabled="checkoption.skip_sub!='请选择'&&checkoption.skip_sub!=''&&checkoption.skip_sub!='0'" :label="checkoption.id" :key="index" class="multiplecheck" :value="checkoption.id">{{checkoption.option_name}}</el-checkbox>
						</el-checkbox-group>
					</span>
				</li>
			</ul>
			<el-button @click="canclerelevance(item)" size="medium">取消</el-button>
			<el-button size="medium" @click="surerelevance">确定</el-button>
		</div>
	</div>
</template>

<script>
	import bus from './eventBus';
	import { jsNumDX } from 'javascripts/utils/index';
	import { Loading } from 'element-ui';
	export default {
		data() {
			return {
				sItem: "", //设置关联的题目
				sOption: [], //选择的 选项
				coptions: [], //选择一道题 对应的有那些选项
				slist: [], // 可选择的题目
				qsItem: "", // 是否设置过 
				qsOption: []
			}
		},
		props: {
			item: {
				type: Object,
				default: {}
			},
			relevanceshow: {
				type: Boolean,
				default: false
			},
			list: {
				type: Array,
				default: []
			},
			qlist: {
				type: Array,
				default: []
			},
			index: {
				type: Number,
				default: 0
			},
			qindex: {
				type: Number,
				default: 0
			},
			relatetype: {
				type: String, // mol 模块关联  moloption 模块题目关联 molcom综合题关联
				default: ""
			}
		},
		methods: {
			canclerelevance(item) {
				this.$emit("canclerelevance", item);
			},
			surerelevance() {
				// 保存 设置过的值  this.sItem, this.sOption, this.qsItem, this.qsOption
				this.saveModelInfo(this.sItem, this.sOption, this.qsItem, this.qsOption);
				this.$emit("surerelevance");
				this.qsItem = this.sItem;
				this.qsOption = this.sOption;
			},
			dArrary(obj, arr) {
				var ary = [];
				for(let km = 0; km < arr.length; km++) {
					if(arr[km] != obj) {
						ary.push(arr[km]);
					}
				}
				return ary;
			},
			saveChangeMethod(sItem, sOption, add) {
				var ares = sItem.split('_');
				var saveModel = {};
				if(ares.length == 2) {
					saveModel = this.list.filter(o => o.id == ares[0])[0].qlist.filter(o => o.id == ares[1])[0];
				}
				if(ares.length == 3) {
					saveModel = this.list.filter(o => o.id == ares[0])[0].qlist.filter(o => o.id == ares[1])[0].qlist.filter(o => o.id == ares[2])[0];
				}
				var forSave = saveModel.option.filter(o => sOption.includes(o.id));
				for(let i = 0; i < forSave.length; i++) {
					forSave[i].related_sub = forSave[i].related_sub.replace("请选择", "");
					var subs = forSave[i].related_sub.trim();
					subs = subs == "0" ? "" : subs;
					subs = subs.length == 0 ? [] : subs.split(',');
					if(add == "add") {
						if(subs.indexOf(this.item.id + "") == -1) {
							subs.push(this.item.id + "");
							forSave[i].related_sub = subs.join(",");
						}
					} else {
						var nsubs = this.dArrary(this.item.id, subs);
						forSave[i].related_sub = nsubs.join(",");
					}

				}
				// 保存设置的值  到数据库   mqitem对象
				this.$post("/Home/Subject/createNewItem", saveModel).then((res) => {
					console.log("add=" + add);
				});
			},
			saveModelInfo(sItem, sOption, qsItem, qsOption) {
				// 先删除  设置题目中  设置过的 关联
				if(qsItem != "" && qsOption.length != 0) {
					this.saveChangeMethod(qsItem, qsOption, "del");
				}
				// 把选择的项 设置 关联
				this.saveChangeMethod(sItem, sOption, "add");
			},
			itemChange(obj) {
//				this.sOption = [];
				this.coptions = this.slist.filter(o => o.id == this.sItem)[0].coptions;
			},
			getMolRelate() {
				for(var loption = 0; loption < this.index; loption++) {
					var mid = this.list[loption].id + "";
					var mname = jsNumDX(this.list[loption].serial_number) + "、" + this.list[loption].mod_name;
					for(var i = 0; i < this.list[loption].qlist.length; i++) {				
						if(this.list[loption].qlist[i].sub_cat == "single") {
							var itemid = this.list[loption].qlist[i].id + "";
							var itemname = this.list[loption].qlist[i].serial_number + ")、" + this.list[loption].qlist[i].title;
							var smodel = {};
							smodel.name = mname + "_" + itemname + "";
							smodel.id = mid + "_" + itemid + "";
							smodel.coptions = this.list[loption].qlist[i].option;
							this.slist.push(smodel);
						}
						if(this.list[loption].qlist[i].sub_cat == "comprehensive") {
							var comitemlist = this.list[loption].qlist[i].qlist.filter(o => o.sub_cat == "single");
							for(var k = 0; k < comitemlist.length; k++) {
								var itemid = this.list[loption].qlist[i].id + "";
								var itemname = this.list[loption].qlist[i].serial_number + ")、" + this.list[loption].qlist[i].title;
								var comid = comitemlist[k].id + "";
								var comname = comitemlist[k].serial_number + "、" + comitemlist[k].title;
								var smodel = {};
								smodel.name = mname + "_" + itemname + "_" + comname;
								smodel.id = mid + "_" + itemid + "_" + comid;
								smodel.coptions = comitemlist[k].option;
								this.slist.push(smodel);
							}
						}
					}
				}
				
			},
			getSelfRelate() {
				var selfList = this.list[this.index].qlist.filter(o => o.serial_number < this.item.serial_number);				
				for(var loption = 0; loption < selfList.length; loption++) {
					var mid = this.list[this.index].id + "";
					var mname = jsNumDX(this.list[this.index].serial_number) + "、" + this.list[this.index].mod_name;
					
					if(selfList[loption].sub_cat == "single") {
						
						var itemid = selfList[loption].id + "";
						var itemname = selfList[loption].serial_number + ")、" + selfList[loption].title;
						var smodel = {};
						smodel.name = mname + "_" + itemname + "";
						smodel.id = mid + "_" + itemid + "";
						smodel.coptions = selfList[loption].option;
						this.slist.push(smodel);
					}
					if(selfList[loption].sub_cat == "comprehensive") {
						var comitemlist = selfList[loption].qlist.filter(o => o.sub_cat == "single");
						for(var k = 0; k < comitemlist.length; k++) {
							var itemid = selfList[loption].id + "";
							var itemname = selfList[loption].serial_number + ")、" + selfList[loption].title;
							var comid = comitemlist[k].id + "";
							var comname = comitemlist[k].serial_number + "、" + comitemlist[k].title;
							var smodel = {};
							smodel.name = mname + "_" + itemname + "_" + comname;
							smodel.id = mid + "_" + itemid + "_" + comid;
							smodel.coptions = comitemlist[k].option;
							this.slist.push(smodel);
						}
					}
				}
				
			},
			getComRelate(){
				if(this.item.sub_cat!='comprehensive'){
					var comList=this.list[this.index].qlist.filter(o => o.sub_cat == "comprehensive");
				var comrelateList=comList.filter(o => o.id == this.item.pid);
				if(comrelateList.length>0){
						var SmallselfList = comList.filter(o => o.id == this.item.pid)[0].serial_number;
						var selfList=comList.filter(o => o.serial_number <SmallselfList);}
				}
				console.log(selfList);
				
//				for(var loption = 0; loption < selfList.length; loption++) {
//					var mid = this.list[this.index].id + "";
//					var mname = jsNumDX(this.list[this.index].serial_number) + "、" + this.list[this.index].mod_name;
//					
//					if(selfList[loption].sub_cat == "single") {
//						
//						var itemid = selfList[loption].id + "";
//						var itemname = selfList[loption].serial_number + ")、" + selfList[loption].title;
//						var smodel = {};
//						smodel.name = mname + "_" + itemname + "";
//						smodel.id = mid + "_" + itemid + "";
//						smodel.coptions = selfList[loption].option;
//						this.slist.push(smodel);
//					}
//					if(selfList[loption].sub_cat == "comprehensive") {
//						var comitemlist = selfList[loption].qlist.filter(o => o.sub_cat == "single" && o.serial_number<this.item.serial_number);
//						for(var k = 0; k < comitemlist.length; k++) {
//							var itemid = selfList[loption].id + "";
//							var itemname = selfList[loption].serial_number + ")、" + selfList[loption].title;
//							var comid = comitemlist[k].id + "";
//							var comname = comitemlist[k].serial_number + "、" + comitemlist[k].title;
//							var smodel = {};
//							smodel.name = mname + "_" + itemname + "_" + comname;
//							smodel.id = mid + "_" + itemid + "_" + comid;
//							smodel.coptions = comitemlist[k].option;
//							this.slist.push(smodel);
//						}
//					}
//				}
				
				
				
			},
			ckchange() {
				if(this.sOption.indexOf(0) != -1) {
					var saveModel = {};
					var ares = this.sItem.split('_');
					if(ares.length == 2) {
						saveModel = this.list.filter(o => o.id == ares[0])[0].qlist.filter(o => o.id == ares[1])[0];
					}
					if(ares.length == 3) {
						saveModel = this.list.filter(o => o.id == ares[0])[0].qlist.filter(o => o.id == ares[1])[0].qlist.filter(o => o.id == ares[2])[0];
					}
					var tipModel = this.slist.filter(o => o.id == this.sItem);
					var result = "请先保存[" + tipModel[0].name + "]的新选项!";
					var saveMidfy = saveModel.option;
					var newArraryDigtal = [];
					this.$alert(result, '提示', {
						confirmButtonText: '确定',
						callback: action => {
							//提交数据库，改变对象
							let loadingInstance = Loading.service({
								lock: true,
								text: '同步更新中……',
								background: 'rgba(0, 0, 0, 0.7)'
							});
							this.$post("/Home/Subject/createNewItem", saveModel).then((res) => {
								var resModel = res.option;
//								return;
								for(var k = 0; k < resModel.length; k++) {
									var nsaveModel = saveMidfy.filter(o => o.order_num == resModel[k].order_num)[0];
									newArraryDigtal.push(nsaveModel.id + "_" + resModel[k].id);
									nsaveModel.id = resModel[k].id;
								}
								this.replaceArrary(this.sOption, newArraryDigtal);
								this.replaceArrary(this.qsOption, newArraryDigtal);
								//更新对象实例
								setTimeout(() => {
									this.$nextTick(() => { // 以服务的方式调用的 Loading 需要异步关闭
										loadingInstance.close();
									});
								}, 10000);
							});
						}
					});
				}
			},
			replaceArrary(arroption, arrrep) {
				for(var k = 0; k < arrrep.length; k++) {
					var arr = arrrep[k].split("_");
					for(var m = 0; m < arroption.length; m++) {
						if(arroption[m] == arr[0]) {
							arroption[m] = arr[1];
						}
					}
				}
			},
			iniPageList() {
				if(this.relatetype == "moloption") {
					this.getMolRelate();
					this.getSelfRelate();
				}
				if(this.relatetype == "mol") {
					this.getMolRelate();
				}
				if(this.relatetype == "molcom") {
					this.getMolRelate();
					this.getComRelate();
//this.getSelfRelate();
				}
			}
		},
		created() {
			// mol 模块关联  moloption 模块题目关联 molcom综合题关联
			//debugger

			this.iniPageList();
			if(this.slist.length > 0) {
				this.sItem = this.slist[0].id;
				this.coptions = this.slist[0].coptions;
			}
//			debugger
			var qsOption = [];
			var qsItem = "";
			var itemid = this.item.id + "";
			for(let qm = 0; qm < this.slist.length; qm++) {
				for(let qk = 0; qk < this.slist[qm].coptions.length; qk++) {
					if(this.slist[qm].coptions[qk].related_sub.split(',').indexOf(itemid) != -1) {
						qsOption.push(this.slist[qm].coptions[qk].id);
						qsItem = this.slist[qm].id;
					}
				}
			}
			if(qsItem != "") {
				if(this.sItem != qsItem) {
					this.coptions = this.slist.filter(o => o.id == qsItem)[0].coptions;
				}
				this.sItem = qsItem;
				this.sOption = qsOption;
				this.qsItem = qsItem;
				this.qsOption = qsOption;
			}
		},
		components: {

		}
	}
</script>

<style scoped="scoped" lang="scss">
	.relevance {
		position: fixed;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, .8);
		top: 0;
		z-index: 200;
		.relevanceitem {
			position: absolute;
			z-index: 300;
			top: 50%;
			left: 50%;
			width: 30%;
			border: 1px dashed #303133;
			background: #fff;
			transform: translate(-50%, -50%);
			padding: 20px 2% 30px;
			.el-button {
				width: 40%;
				display: inline-block;
				margin-top: 30px;
				&:nth-of-type(1) {
					margin-right: 10%;
					margin-left: 5%;
				}
			}
			.relevanceitemcontent {
				width: 100%;
				margin: 0 auto;
				border-top: 1px solid #303133;
				border-left: 1px solid #303133;
				float: left;
				li {
					border-bottom: 1px solid #303133;
					border-right: 1px solid #303133;
					text-align: center;
					float: left;
					width: 100%;
					>span {
						&:nth-of-type(1) {
							background: #409EFF;
						}
					}
					span {
						display: inline-block;
						padding: 8px 0;
						&:nth-of-type(1) {
							width: 30%;
							border-right: 1px solid #303133;
						}
						width: 69%;
						float:left;
					}
				}
			}
		}
	}
</style>