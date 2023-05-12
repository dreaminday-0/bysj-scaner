<template>
	<view class="content">
		<uni-section title="请选择教室" type="line">
			<uni-data-picker v-model="select" placeholder="请选择教室"  popup-title="请选择教室" :localdata="roomInf" >
			    </uni-data-picker>
		</uni-section>
		<uni-section title="输入管理员账号" type="line">
			<uni-easyinput placeholder="输入管理员账号" v-model="name"></uni-easyinput>
		</uni-section>
		<uni-section title="输入管理员密码" type="line">
			<uni-easyinput placeholder="输入管理员密码" v-model="password"></uni-easyinput>
		</uni-section>
		<view class="btnView">
			<button @click="login()" type="default">确认登录</button>
			<button @click="scanBtn()" :disabled="disable" type="default">开始扫码</button>
		</view>
			
	</view>
</template>

<script>
	import url from '../../urlConfig.js'
	import md5 from '../../util/md5.js'
	export default {
		data() {
			return {
				select:'',
				roomInf:[],
				colleges:[],
				rooms:[],
				password:'',
				name:'',
				disable:true
			}
		},
		onLoad() {
			this.initInf()
		},
		methods: {
			scanBtn(){
				let that=this;
				uni.scanCode({
					success(res) {
						console.log(res.result)
						that.submitScan(res.result);
					}
				})
			},
			submitScan(uid){
				let that=this;
				console.log(uid);
				uni.request({
					url:url+'/admin/signIn?roomId='+that.select+"&userId="+uid,
					header:{
						"Authorization":uni.getStorageSync("token")
					},
					success(res) {
						if(res.data.res=='1'){
							uni.showToast({
								icon:'success',
								title:"成功",
								duration:1000,
								complete() {
									that.scanBtn();
								}
							})
						}else if(res.data.res=='2'){
							uni.showToast({
								icon:'error',
								title:"签到失败 房间错误",
								duration:1000,
								complete() {
									that.scanBtn();
								}
							})
						}
					}
				});
			}
			,
			getColleges(){
				let that=this;
				return new Promise((resolve,reject)=>{
					uni.request({
						url:url+'/login/getColleges',
						success(res) {
							that.colleges = res.data;
							resolve('success');
						}
					})
				})
			},
			getRooms(){
					let that = this;
				return new Promise((resolve,reject)=>{
					 uni.request({
						url:url+'/login/getRooms',
						success(res) {
							
							that.rooms = res.data;
							resolve('success');
						}
					})
				})
				
			},
			async initInf(){
				await this.getRooms();
				await this.getColleges();
				this.colleges.forEach(item=>{
					let children=this.rooms.filter(room=>room.whichCollege===item.value);
					const rooms = JSON.parse(JSON.stringify(children).replace(/name/g,"text").replace(/roomId/g,"value"));
					this.roomInf.push({
						text:item.text,
						value:item.value,
						children:rooms
					})
				});
				console.log(this.roomInf)
			},
			login(){
				let that=this;
				uni.request({
					url:url+'/admin/login',
					method:'POST',
					data:{
						name:that.name,
						password:md5(that.password)
					},
					success(res) {
						if (res.data.status==='0'){
							uni.showToast({
								icon:'error',
								title:'登录失败',
								duration:1000
							})
						}else {
							uni.showToast({
								icon:'success',
								title:'登录成功',
								duration:1000
							});
							uni.setStorageSync("token",res.data.token);
							that.disable=false;
						}
					}
				})
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
	.btnView{
		display: flex;
		align-items: center;
	}
</style>
