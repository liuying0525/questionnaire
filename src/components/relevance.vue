<template>
	<div v-show="relevanceshow" class="relevance">
		<div class="relevanceitem">
			<p>关联逻辑设置：</p>
			<ul class="relevanceitemcontent">
				<li>
					<span>当前选项：</span>
					<span>{{item.serial_number}}{{item.title}}</span>
				</li>
				<li>
					<span>关联题目：</span>
					<span>	
						<el-select v-model="sItem" placeholder="请选择" @change="itemChange">
							<el-option v-for="(opitem,oindex) in slist" :key="oindex"  :label="opitem.serial_number+opitem.title"  :value="opitem.id"></el-option>
						</el-select>
					</span>
				</li>
				<li>
					<span>题目选项：</span>
					<span>					
					<el-checkbox-group v-model="sOption">
						<el-checkbox v-for="(checkoption,index) in coptions" :disabled="checkoption.skip_sub!='请选择'&&checkoption.skip_sub!=''" :label="checkoption.id" :key="index" class="multiplecheck" :value="checkoption.id">{{checkoption.option_name}}</el-checkbox>
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
			domains: {
				type: Array,
				default: []
			},
			qlist: {
				type: Array,
				default: []
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
			saveModelInfo(sItem, sOption, qsItem, qsOption) {
				var mqitem = {};
				// 先删除  设置题目中  设置过的 关联
				if(qsItem != "" && qsOption.length != 0) {
					mqitem = this.qlist.filter(o => o.id == qsItem)[0];
					//var mqoption = mqitem.option.filter(o => qsOption.indexOf(o.id) != -1);
					for(var i = 0; i < qsOption.length; i++) {
						for(var k = 0; k < mqitem.option.length; k++) {
							if(qsOption[i] == mqitem.option[k].id) {
								var subs = mqitem.option[k].related_sub.trim();
								subs = subs == "0" ? "" : subs;
								subs = subs.length == 0 ? [] : subs.split(',');
								var nsubs = this.dArrary(this.item.id, subs);
								mqitem.option[k].related_sub = nsubs.join(",");
							}
						}
					}
					this.$post("/Home/Subject/createNewItem", mqitem).then((res) => {
						console.log("del");
					});
				}
				// 把选择的项 设置 关联
				var nmqitem = this.qlist.filter(o => o.id == sItem)[0];
				for(let k = 0; k < sOption.length; k++) {
					for(let i = 0; i < nmqitem.option.length; i++) {
						if(sOption[k] == nmqitem.option[i].id) {
							if(nmqitem.option[i].related_sub.trim() == "0") {
								nmqitem.option[i].related_sub = "";
							}
							var subs = nmqitem.option[i].related_sub.trim();
							subs = subs == "0" ? "" : subs;
							subs = subs.length == 0 ? [] : subs.split(',');
							if(subs.indexOf(this.item.id + "") == -1) {
								subs.push(this.item.id + "");
								nmqitem.option[i].related_sub = subs.join(",");
							}
						}
					}
				}
				// 保存设置的值  到数据库   mqitem对象
				this.$post("/Home/Subject/createNewItem", nmqitem).then((res) => {
					console.log("add");
				});
			},
			itemChange(obj) {
				//选择不同的题目  显示  题目中的选项 
				this.coptions = this.slist.filter(o => o.id == this.sItem)[0].option;
			}
		},
		created() {

			//筛选当前题目前面的 单选题
			var ck = this.qlist.filter(o => o.sub_cat == 'single' && o.serial_number < this.item.serial_number);
			this.slist = ck;
			if(this.slist.length > 0) {

				//初始化默认
				this.sItem = this.slist[0].id;
				this.coptions = this.slist.filter(o => o.id == this.sItem)[0].option;
				//debugger
				//获取设置过的值    查找是否设置过选项 回显
				var itemid = this.item.id;
				var qsOption = [];
				var qsItem = "";
				for(let qm = 0; qm < ck.length; qm++) {
					for(let qk = 0; qk < ck[qm].option.length; qk++) {
						if(ck[qm].option[qk].related_sub.indexOf(itemid) != -1) {
							qsOption.push(ck[qm].option[qk].id);
							qsItem = ck[qm].id;
						}
					}
				}
				if(qsItem != "") {
					if(this.sItem != qsItem) {
						//显示对应题目的选项
						this.coptions = this.slist.filter(o => o.id == qsItem)[0].option;
					}
//					debugger
					this.sItem = qsItem;
					this.sOption = qsOption;
					this.qsItem = qsItem;
					this.qsOption = qsOption;
				}
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