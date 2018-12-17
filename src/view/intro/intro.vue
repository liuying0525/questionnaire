<template>
	<div class="intro">
		
		<h4>请输入公司简介内容：</h4>
		<el-row  type="flex" justify="center" >
			<el-col :span="18">
		<el-input
  type="textarea"
  :rows="10"
  size="small"
 
  placeholder="请输入公司简介的内容"
  v-model="textarea">
</el-input>
</el-col>
</el-row>
<el-row type="flex" justify="center" class="introbtn">
	<el-col :span="6" :offset="6">
	<el-button @click="save">保存</el-button>
		</el-col>
	<el-col :span="6" :offset="6"> 
	<el-button>取消</el-button>
	</el-col>
</el-row>
	</div>
</template>

<script>
	export default{
		data(){
			return{
				textarea:"",
				contentid:""
			}
		},
		methods: {
		save(){
				this.$post("/Home/Company/edit",{"id":this.contentid,"introduction":this.textarea}).then((res)=>{
						this.$message({
							type: 'success',
							message: '公司简介保存成功！'
						});
				});
		},
		getInfo(){
				this.$post("/Home/Company/index",{}).then((res)=>{
					this.contentid=res.id;
					this.textarea=res.introduction;
				});
		}
			
			
		},
		mounted() {

		},
		created(){
			this.getInfo();
		},
	}
</script>

<style>
	.intro{
		margin-top:58px;
		
	}
.intro	h4{
			margin-left:12.5%;
			line-height: 50px;
		}
		.introbtn{
			margin-top:50px;
		}
</style>
<style scoped="scoped" lang="scss">
	.el-main{
		margin-top:0;
	}
</style>