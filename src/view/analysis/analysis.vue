<template>
	<div>
		<div class="editTemContain">
			<div>
				<el-row type="flex" justify="space-between" class="conTop">
					<el-col :span="12">
						<!--<el-button class="analysisicon">分析&下载</el-button>-->
					</el-col>
					<el-col :span="12" class="toprightbt">
						<!--<el-button class="downloadicon" bindtap="downreport">下载此报告</el-button>
						<el-button class="lasticon" bindtap="downoriginal">下载原始问卷</el-button>-->
						<el-button class="downloadicon" @click.prevent="downloadanswer" :href="report">下载原始答案</el-button>
						
						
						<el-button class="downloadicon" @click.prevent="downreport" :href="report">下载此报告</el-button>
						<el-button class="lasticon" @click.prevent="downoriginal" :href="original">下载原始问卷</el-button>
					</el-col>
				</el-row>
				<div class="conBottom">
					<div class="conBottomT">
						<h6>{{questiontitle}}</h6>
						<p v-text="contentText"></p>
					</div>

					<el-collapse v-model="activeNames" v-for="(item,index) in modList" :key="item.id" @change="handleChange">

						<el-collapse-item :title="item.mod_name" :name="index">
							<div class="collapseinner">
								<el-col :span="24" v-if="item.mod.length!=0" v-for="(mitem,mindex) in item.mod" :key="mitem.id">
									<!--<span>{{qitem.modorder}}、{{qitem.itemmodname}}</span>-->
									<!--<span v-if="mitem.serial_number==mindex">{{mitem.serial_number}}、{{mitem.mod_name}}</span>-->
								</el-col>
								<el-row v-for="(qitem,qindex) in item.item" :key="qitem.id" v-bind:class="qitem.modorder?'bordernone':''">
									<span v-if="qitem.itemmodname&&qitem.order==1" class="comvetitle">{{Number(qitem.modorder)}}、{{qitem.itemmodname}}</span>
									<el-col :span="24" class="itemtitle"><span>{{qitem.order}}</span><span>{{qitem.itemmodname?")":"、"}}</span><span>{{qitem.title}}：</span><span class="typetip">【{{qitem.cat}}】</span></el-col>

									<el-col :span="24" v-if="qitem.sub_cat=='single'||qitem.sub_cat=='multiple'">
										<el-table :data="qitem.option" border style="width: 100%" :summary-method="getSummaries" show-summary>
											<el-table-column prop="option_name" label="选项" width="180">
											</el-table-column>

											<el-table-column prop="num" label="小计" width="180">
											</el-table-column>
											<el-table-column label="比例">
												<!--<template slot-scope="scope" v-if="scope.row.date!='本题有效填写人次'">-->
												<template slot-scope="scope">
													<el-progress :percentage="parseInt(scope.row.percent*10000)/100||0" style=""></el-progress>
												</template>
											</el-table-column>

										</el-table>
									</el-col>
									<el-col :span="23" :offset="1" v-else>
										<el-button size="medium" @click="tagItem(qitem)" class="checkmy">查看详细信息</el-button>
										<el-table :data="qitem.resultList" border style="width: 100%" v-show="!!qitem.itemshow">
											<el-table-column prop="rownum" label="序号" width="80"></el-table-column>
											<el-table-column prop="answer_time" label="日期" width="180"></el-table-column>
											<el-table-column label="详细内容" v-if="qitem.sub_cat=='uploadimg'">
												<template slot-scope="scope">
												<el-button v-if="imgitem" v-for="(imgitem,imgindex) in qitem.resultList[scope.$index].result"  @click="open4(imgitem)"  :key="imgitem.id">
													<img :src="imgitem"  class="uploadimgList"/>
												</el-button>
												</template>
											</el-table-column>
											<el-table-column label="详细内容" v-if="qitem.sub_cat=='signature'" width="210">
												<template slot-scope="scope">
													<el-button v-if="qitem.resultList[scope.$index].result"   @click="open5(qitem,scope.$index)">
													<img :src="qitem.resultList[scope.$index].result"  class="signature"/>
													</el-button>
												</template>
											</el-table-column>
											<el-table-column label="详细内容" v-if="qitem.sub_cat=='loCation'" prop="result">
											</el-table-column>
											<el-table-column prop="result" label="详细内容" v-else>
											</el-table-column>

										</el-table>

									</el-col>

								</el-row>

							</div>

						</el-collapse-item>
					</el-collapse>
				</div>
			</div>
			<div v-show="jumpshow" class="jump">
			<div class="jumpitem">

				<p>下载问卷</p>
				<template>
					<el-radio v-model="ckRadio" label="1" class="establish">下载原答案版</el-radio>
					<el-radio v-model="ckRadio" label="2" class="establish">下载文字描述版</el-radio>
				</template>
		
				<el-button @click="canclejump" size="medium">取消</el-button>
				<el-button size="medium" @click="submit">确定</el-button>
			</div>
		</div>
			
			
			
		</div>
		
	</div>
</template>

<script>
	import headTop from 'view/head/headTop.vue';
	import topic from 'components/topic.vue';
	import { jsNumDX } from 'javascripts/utils/index';
	import { Message } from "element-ui";
	import JSZip from "jszip";
	import axios from "axios";
	

	import { ofill, osingle, omultiple, omultistage, ouploadimg, oloCation, ofractions, ocomprehensive } from "components/itemType";

	export default {
		data() {
			return {
				contentText: '',
				activeNames: ['1'],
				questiontitle: "",
				quesId: "",
				modList: [],
				quesList: [],
				report:'',
				original:'',
				resultList: [],
				jumpshow:false,
				ckRadio: '1',
				tableData: [{
					date: '男',
					name: '0',
					address: 50

				}, {
					date: '女',
					name: '0',
					address: 0
				}, {
					date: '本题有效填写人次',
					name: '2',

				}]

			}
		},
		methods: {
			handleChange(val) {

			},
			getItemOptions(itemList) {
				let resList = [];
				Array.from(itemList, (obj, index) => {
					let sub_cat = obj.sub_cat;
					obj.resultList = [];
					switch(sub_cat) {
						case "multiple":
							{
								let multiple = JSON.parse(JSON.stringify(obj));
								for(var km in multiple.option) {
									multiple.option[km].effective = obj.effective;
								}
								resList.push(multiple);
							}
							break;
						case "single":
							{
								let single = JSON.parse(JSON.stringify(obj));
								for(var km in single.option) {
									single.option[km].effective = obj.effective;
								}
								resList.push(single);
							}
							break;

						default:
							{
								resList.push(obj);
							}
							break;
					}

				}, this)
				return resList;

			},
			open4(imgitem){
			
				   this.$alert('<img src='+imgitem+' class="uploadimgListview"/>', '图片预览', {
			          dangerouslyUseHTMLString: true
			        })   
			        .catch(action => {
         
          				});
			},
			open5(qitem,index){
			
				   this.$alert('<img src='+qitem.resultList[index].result+' class="signatureview"/>', '图片预览', {
			          dangerouslyUseHTMLString: true
			        })   
			        .catch(action => {
         
          				});
			},
			getSummaries(param) {
				const {
					columns,
					data
				} = param;
				const sums = [];
				var lyconut = data[0].effective || "";
				columns.forEach((column, index) => {
					if(index === 0) {
						sums[index] = '本次有效填写人次';
						return;
					}
					if(index === 1) {
						sums[index] = lyconut;
						return;
					}
				});
				return sums;
			},
			tagItem(item) {
				var _this = this;
				let type = item.sub_cat;
				item.itemshow = !item.itemshow;
				this.$post("/Home/Analysis/detail", {
					id: item.id
				}).then((res) => {
					if(item.sub_cat == 'uploadimg'||item.sub_cat=='signature') {
						
						var imgLoad = res;
						res.forEach((obj, index) => {
							item.sub_cat == 'uploadimg'?(obj.result = JSON.parse(obj.result)):(obj.result = obj.result);
						});
						item.resultList = imgLoad;
					
					} else if(item.sub_cat == 'loCation') {
						var obj = res || [];
						res.forEach((obj, index) => {
							obj.result = JSON.parse(obj.result).addresname + "," + JSON.parse(obj.result).locationName;
						});
						item.resultList = obj; //objres.addresname+obj.locationName;
					} else {
						item.resultList = res;
						
					}
				})
			},
			downreport(){
				var _this=this;
				this.$post("/Home/Download/analysis", {
					id: this.$route.query.questionId
				}).then((res) => {
					
					location.href=process.env.http.root.replace("index.php","")+"/"+res.path;
					
					
					//_this.report=res.path;
					
				})
			},
				downreport(){
				var _this=this;
				this.$post("/Home/Download/analysis", {
					id: this.$route.query.questionId
				}).then((res) => {
					
					location.href=process.env.http.root.replace("index.php","")+"/"+res.path;
					
					
					//_this.report=res.path;
					
				})
			},
			downoriginal(){
			this.jumpshow=true;
			},
			canclejump(){
				this.jumpshow=false;
			},
			submit(){
				var _this=this;
				this.$post("/Home/Download/excel", {
					id: this.$route.query.questionId,
					status:this.ckRadio
				}).then((res) => {
					location.href=process.env.http.root.replace("index.php","")+"/"+res.path;
				})
				this.jumpshow=false;
			},

				downloadanswer(){
				var _this=this;
					this.$confirm('下载答卷需要耗时很久，您确定下载么?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
							this.$post("/Home/DLOriginal/original", {
					id: this.$route.query.questionId
				},{timeout: 100000}).then((res) => {
			
				if(res.paths.length>1){
					for(var i=0;i<res.paths.length;i++){
						window.open(process.env.http.root.replace("index.php","")+"/"+res.paths[i])
					}
				}else{
					location.href=process.env.http.root.replace("index.php","")+"/"+res.paths;
				}
//					location.href=process.env.http.root.replace("index.php","")+"/"+res.paths;
					
					
				})
				}).catch(() => {});
				
				
				
				
	
			}
		},
		mounted: function() {

		},
		components: {
			headTop,
			topic
		},
		created() {
			this.quesId = this.$route.query.questionId;
			this.$post("/Home/Analysis/itemList", {
				id: this.quesId
			}).then((res) => {
				this.contentText = res.description || "";
				this.questiontitle = res.sub_name || "";
				let modlist = res.mod;
				let activelist = [];
				for(var k = 0; k < modlist.length; k++) {
					var option = {};
					activelist.push(k);
//					debugger
					option.mod_name =modlist[k].mod_name;
					option.qtitle = this.quesList.length + 1 + '、';
					option.id = modlist[k].id;
					option.sortId = modlist[k].order_num;
					option.qlist = [];
					option.pid = modlist[k].pid;
					if(modlist[k].item.length != 0) {
						let opList = this.getItemOptions(modlist[k].item);
						option.qlist = modlist[k].item;
						modlist[k].item = opList;
						for(var n = 0; n < modlist[k].item.length; n++) {
							modlist[k].item[n].modorder=modlist[k].item[n].order;
						}
						
					}
					if(modlist[k].mod && modlist[k].mod.length != 0) {
						
						for(var j = 0; j < modlist[k].mod.length; j++) {
							var sortList=[];
							for(var v = 0; v < modlist[k].mod[j].item.length; v++) {
								modlist[k].mod[j].item[v].itemmodname = modlist[k].mod[j].mod_name;
//								debugger
							modlist[k].mod[j].item[v].modorder = modlist[k].mod[j].serial_number;
								modlist[k].item.push(modlist[k].mod[j].item[v]);
								modlist[k].item.sort(function(a,b){
										return a.modorder-b.modorder;
								})
								
							}

						}
						let opList2 = this.getItemOptions(modlist[k].item);
						option.qlist = modlist[k].item;
						//debugger
						modlist[k].item = opList2;
					}

					for(var j = 0; j < option.qlist.length; j++) {
						option.qlist[j].itemshow = false;
					}
					modlist[k].mod_name = jsNumDX(k + 1) + modlist[k].mod_name;
					//.debugger
					this.modList.push(modlist[k]);
					//var modList = this.modList;
					

				}
				this.activeNames = activelist;

			});
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
		font-weight: 600;
	}
	
	.conBottom .el-collapse-item__content {
		padding: 10px 0;
		border-top: 1px solid #ebeef5;
	}
	
	.el-table th>.cell {
		text-align: center;
	}
	.signatureview{
		    width: 30%;
       transform: rotate(90deg) translate(-25%,-50%);
    margin-bottom: -20%;
	}
	.editTemContain .el-table--enable-row-transition .el-table__body td .el-button{
		border:none
	}
.uploadimgListview{
		    max-width: 100%;
   
	}
	.editTemContain .el-progress-bar__inner{
		left:0;
	}
	.editTemContain .el-progress-bar{
		width: 95%;
		padding-left:20px;
	}
	.el-table td div.el-progress{text-align: left;}
	.editTemContain .el-table .cell{
		
		padding-left:0;
	}
</style>
<style scoped="scoped" lang="scss">
	.el-main {
		background-color: #f3f3f3;
	}
	
	.editTemContain {
		padding: 68px 120px 0;
		background-color: #f3f3f3;
		>div {
			padding: 0 30px;
		}
	}
	
.conTop {
		padding: 15px 0;
		color: #fff;
		font-size: 14px;
		.el-button {
			border: none;
			text-align: center;
			color: #fff;
			padding: 14px 26px;
			padding-left: 50px;
			&.analysisicon {
				background: #ffbc1b url(../../statics/images/analysisicon.png) 18px center no-repeat;
			}
			&.downloadicon {
				background: #005ad4 url(../../statics/images/downloadicon.png) 18px center no-repeat;
			}
			&.lasticon {
				border: 1px solid #005ad4;
				color: #005ad4;
				padding: 14px 25px;
				padding-left: 14px;
			}
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
		>p {
			color: #666;
			line-height: 30px;
			font-size:14px;
			font-weight: 600;
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
		position: relative;
	}
	
	.el-collapse {
		border: none;
	}
	
	.toprightbt {
		text-align: right;
	}
	
	.collapseinner {
		padding: 22px;
		.el-row {
			padding-bottom: 30px;
			border-bottom: 1px dashed #e4e4e4;
			>.el-col:nth-of-type(1) {
				margin-bottom: 25px;
			}
		}
	}
	
	.el-table,
	.el-table th>.cell {
		text-align: center;
	}
	
	.collapseinner .el-row {
		padding-top: 30px;
	}
	
	.uploadimgList {
		width:50%;
		height:auto;
	}
	
	.bordernone:not(:last-child) {
		border-bottom: none !important;
	}
	
	.comvetitle {
		display: block;
		margin: 10px 0 45px;
	}
	.checkmy{
		margin-bottom:20px;
	}
	
	
		.jump {
		position: fixed;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, .3);
		top: 0;
		z-index: 200;
		p {
			padding: 10px 0 20px;
		}
		/*.el-radio__inner {
			background-color: #333;
		}
		.el-radio {
			color: #fff;
		}*/
		.jumpitem {
			position: absolute;
			z-index: 300;
			top: 50%;
			left: 50%;
			width: 30%;
			background: #eef9fb;
			color: #606266;
			/*border:1px solid #409EFF;*/
			box-shadow:0 3px 5px rgba(64,158,255,.3);
			border-radius: 4px;
			transform: translate(-50%, -50%);
			padding: 20px 2% 30px;
			text-align: center;
			.el-button {
				width: 40%;
				display: inline-block;
				margin-top: 30px;
				&:nth-of-type(1) {
					margin-left: 5%;
				}
			}
		
		}
	}
	.itemtitle{
		font-weight: bold;
	}
	.conBottomT h6{font-size:20px;}
	.signature{
		width: 20%;
		height: auto;
		transform: rotate(90deg);
	}
	.el-message-box__message p{
		    height: 200px;
    margin: 0 auto;
    text-align: center;
    line-height: 200px;
	}

</style>