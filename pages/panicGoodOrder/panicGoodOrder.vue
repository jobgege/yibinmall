<template>
	<view class="main">
		<view class="item">
			<view class="head">
				<view class="container">
					<img :src="details.goods.goods_main_picture" alt="商品图片" class="goodImg">
					<view class="goodText">
						<view class="goodDetail">{{details.goods.goods_name}}</view>
						<view class="goodPrice">共青价：<span style="color: red; font-weight: bold;" >鲜豆{{details.panicBuyingGoods.panic_buying_price}}</span></view>
					</view>
				</view>
				<view class="deliveryWay">
					<view class="way">配送方式：</view>
					<view class="deliver">线下商家配送（ 送货上门 ）</view>
				</view>
			</view>
			<view class="body" @click="change">
				<view class="option">
					<view>收货人</view>
					<view>{{person.real_name}}</view>
				</view>
				<view class="option">
					<view>收货电话</view>
					<view>{{person.tel}}</view>
				</view>
				<view class="option">
					<view>收货地址</view>
					<view>{{person.address}}</view>
				</view>
				<view class="option">
					<view>货物数量</view>
					<view>{{person.number}}</view>
				</view>
			</view>
			<view class="foot">
				<view class="bean">
					<view>商品鲜豆</view>
					<view>{{details.panicBuyingGoods.panic_buying_price}}</view>
				</view>
				<view class="star">
					<view>会员星级</view>
					<view>{{details.goods.star}}星</view>
				</view>
				<view class="flashTime">
					<view>市场价</view>
					<view>{{details.goods.market_price}}元</view>
				</view>
				<view class="sum"><span style="color: red; font-weight: bold;" >鲜豆{{person.number * details.panicBuyingGoods.panic_buying_price}}</span>合计： </view>
			</view>
		</view>
		<view class="pay">
			<view style="color: red; font-weight: bold;">
				鲜豆{{person.number * details.panicBuyingGoods.panic_buying_price}}
			</view>
			<view class="btn"><button class="btn" @click="buy">立即支付</button></view>
		</view>
		
		<!-- 弹窗 -->
		<view v-if="showPop" class="modal"> 
			<!-- <form @submit="formSubmit"> -->
				<view class="inputItem">
					收货人<input v-model="person.real_name" class="input" type="text" name="name" placeholder="请输入收货人姓名" placeholder-style="font-size:26rpx;color:grey;">
				</view>
				<view class="inputItem">
					收货电话<input v-model="person.tel" class="input" type="text" name="phone" placeholder="请输入收货电话" placeholder-style="font-size:26rpx;color:grey;">
				</view>
				<view class="inputItem">
					收货地址<input v-model="person.address" class="input" type="text" name="address" placeholder="请输入收货地址" placeholder-style="font-size:26rpx;color:grey;">
				</view>
				<view class="inputItem">
					货物数量<input v-model="person.number" class="input" type="text" name="phone" placeholder="请输入货物数量" placeholder-style="font-size:26rpx;color:grey;">
				</view>
				<button @click="closeModal()" class="login" >确定</button>
			<!-- </form> -->
		</view>
		
	</view>
</template>

<script>
import getToken from '../../publicAPI/getToken.js'
import updatePersonMsg from '../../publicAPI/updataPersonMsg.js'
import {baseURL} from '../../publicAPI/baseData.js'
export default {
	data() {
		return {
			showPop: false,
			details:{},
			person: getApp().globalData.UserInfo,
			post:{
				order_user_id: undefined,
				store_id: undefined,
				goods_id: undefined,
				coupons_id: undefined,
				number: undefined,
				order_status: undefined,
				consignee_name: undefined,
				consignee_phone: undefined,
				consignee_address: undefined,
				deliver_type: undefined,
				order_time: undefined,
				deliver_time: undefined,
			},
		}
	},
	onLoad(option) {
		if (option.details){
			// decodeURIComponent 解密传过来的对象字符串
			this.details = JSON.parse(decodeURIComponent(option.details));
			console.log(this.details)
		}
		this.person.number = 1
		console.log(getApp().globalData.UserInfo)
		
	},
	methods: {
		buy(){
			if (this.person.address && this.person.tel && this.person.real_name && this.person.star >= this.details.goods.star && this.person.beans >= this.person.number * this.details.panicBuyingGoods.panic_buying_price){
				this.post.order_user_id = this.person.id
				this.post.store_id = this.details.goods.store_id
				this.post.goods_id = this.details.goods.goods_id
				this.post.coupons_id = this.details.goods.coupons_id
				this.post.number = this.person.number
				this.post.order_status = "已支付"
				this.post.consignee_name = this.person.real_name
				this.post.consignee_phone = this.person.tel
				this.post.consignee_address = this.person.address
				this.post.deliver_type = "线下厂商配送"
				this.post.order_time = new Date().getTime()
				this.post.deliver_time = undefined
				
				console.log(this.post)
				let app = getApp()
				let poll = {
					user_id: this.person.id,
					type: 1,
					thingsId: this.details.goods.goods_id,
				}
				//getToken获取token，第一个参数是成功的回调，第二个参数是失败的回调
				getToken(
				res => {
					//将token存入全局变量中
					let app = getApp()
					app.globalData.Authorization = res.data
					//发送购买请求
					uni.request({
						url: baseURL + '/pb_orders',
						method: "POST",
						data: this.post,
						header: {
							'Authorization':"Bearer "+app.globalData.Authorization,
						},//请求头
						dataType: "json",
						sslVerify: false, 
						success: res => {
							console.log(res)
							//轮询抢购结果
							if (res.data.code == 200){
								uni.showLoading({
									title: res.data.message
								});
								let timer = setInterval(()=>{
									uni.request({
										url: baseURL + '/pb_orders/result/'+poll.user_id+'/'+poll.type+'/'+poll.thingsId,
										method: "GET",
										// data: poll,
										header: {
											'Authorization':"Bearer "+app.globalData.Authorization,
										},//请求头
										dataType: "json",
										sslVerify: false, 
										success: res => {
											console.log(res)
											uni.hideLoading()
											if (res.data.code == 200){
												uni.showModal({
													title: '抢购成功',
													// content: '是否查看订单详情',
													success: function (res) {
														updatePersonMsg()//更新鲜豆信息
														if (res.confirm) {
															setTimeout(()=>{																
																uni.navigateTo({
																	url: '../mall/mall'
																})
															},500)
														} else if (res.cancel) {
															setTimeout(()=>{
																uni.navigateTo({
																	url: '../mall/mall'
																})
															},500)
														}
													}
												});
												clearInterval(timer)
											}else if(res.data.code == 500){
												uni.hideLoading()
												uni.showToast({
													icon: 'none',
													title: res.data.message
												});
												clearInterval(timer)
											}
										},
										fail: err => {
											uni.hideLoading()
											uni.showToast({
												icon: 'none',
												title: "订单信息获取失败，请重试！"
											});
											uni.hideLoading()
										}
									})
								},1000)
							}else{
								uni.showToast({
									icon: 'none',
									title: res.data.message
								});
							}
							
						},
						fail: err => {
							uni.showToast({
								icon: 'none',
								title: "订单信息发送失败，请重试！"
							});
						}
					})
				},
				err => {
					uni.showToast({
						icon: 'none',
						title: "获取token失败，请重试！"
					});
				}
				)
				
			}else if(this.person.star < this.details.star){
				uni.showToast({
					icon: 'none',
					title: "用户星级不足"
				});
			}else if (this.person.beans < this.person.number * this.details.panicBuyingGoods.panic_buying_price){
				uni.showToast({
					icon: 'none',
					title: "用户鲜豆不足"
				});
			}
			else{
				console.log(this.person)
				uni.showToast({
					icon: 'none',
					title: "请检查收货信息是否正确"
				});
			}
			// uni.navigateTo({
			// 	url:'../mall/mall'
			// })
		},
		change(){
			this.showPop = true
		},
		closeModal(){
			this.showPop = false
		},
	}
}
</script>

<style>
	.main{
		height: 100vh;
		background-color: rgb(245,245,245);
	}
	
	.item{
		height: 70vh;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}
		
	.head{
		height: 18vh;
		padding: 5vw;
		background-color: white;
	}
	
	.container{
		display: flex;
		flex-wrap: wrap;
		flex-direction: row;
		justify-content: space-between;
		height: 30vw;
	}
		
	.goodImg{
		height: 20vw;
		width: 20vw;
		display: inline;
		border-radius: 5%;
	}
	
	.goodText{
		width: 60vw;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}
	
	.goodDetail{
		width: 60vw;
		height: 13vw;
		overflow: hidden;  
		font-weight: bold;
		font-size: 10px;
	}
	
	.goodPrice{
		width: 60vw;
		height: 12vw;
		font-size: 10px;
	}
	
	.deliveryWay{
		font-size: 10px;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.body{
		font-size: 10px;
		padding: 5vw;
		background-color: white;
	}
	
	.option{
		margin: 1vh 0;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.foot{
		font-size: 10px;
		padding: 5vw;
		background-color: white;
	}
	
	.bean{
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.star{
		margin: 1vh 0;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.flashTime{
		margin: 1vh 0;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.sum{
		margin: 1vh 0;
		display: flex;
		flex-direction: row-reverse;
	}
	
	.pay{
		width: 100vw;
		height: 4vh;
		padding: 5vw;
		position: fixed;
		bottom: 0;
		background-color: white;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
	
	.btn{
		width: 28vw;
		height: 3vh;
		margin-right: 10vw;
		background-color: red;
		color: #FFFFFF;
		line-height: 3vh;
		text-align: center;
		border-radius: 12px;
	}
	
	/* 弹窗 */
	.modal{
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%,-50%);
		background-color: white;
		width: 70vw;
		border: 1px rgb(195,195,195) solid;
		padding: 1vh 3vw;
	}
	.inputItem{
		display: flex;
		flex-direction: row;
		font-size: 12px;
		margin:2vh 0;
		vertical-align: sub;
	}
	.input{
		margin-left: 3vw;
		border-bottom: 1px rgb(195,195,195) solid;
	}
	
</style>

<!-- uni.request({
						url: 'http://yibinmall.chenglee.top:8080/get_token',//开发者服务器接口地址
						method: "POST",
						data: msg,//请求的参数
						dataType: "json",
						sslVerify: false, 
						success: res => {
							//将token存入全局变量中
							let app = getApp()
							app.globalData.Authorization = res.data
							//发送购买请求
							uni.request({
								url: 'http://yibinmall.chenglee.top:8080/pb_orders',
								method: "POST",
								data: this.post,
								header: {
									'Authorization':"Bearer "+app.globalData.Authorization,
								},//请求头
								dataType: "json",
								sslVerify: false, 
								success: res => {
									console.log(res)
									uni.showLoading({
										title: res.data.message
									});
									//轮询抢购结果
									if (res.data.code == 200){
										uni.showLoading({
											title: res.data.message
										});
										let timer = setInterval(()=>{
											uni.request({
												url: 'http://yibinmall.chenglee.top:8080/pb_orders/result/'+poll.user_id+'/'+poll.type+'/'+poll.thingsId,
												method: "GET",
												// data: poll,
												header: {
													'Authorization':"Bearer "+app.globalData.Authorization,
												},//请求头
												dataType: "json",
												sslVerify: false, 
												success: res => {
													console.log(res)
													uni.hideLoading()
													if (res.data.code == 200){
														uni.showModal({
															title: '抢购成功',
															content: '是否查看订单详情',
															success: function (res) {
																if (res.confirm) {
																	console.log('用户点击确定');
																} else if (res.cancel) {
																	console.log('用户点击取消');
																}
															}
														});
														clearInterval(timer)
													}else if(res.data.code == 500){
														console.log("抢购失败")
														clearInterval(timer)
													}
												},
												fail: err => {
													uni.showToast({
														icon: 'none',
														title: "订单信息获取失败，请重试！"
													});
													uni.hideLoading()
												}
											})
										},1000)
									}
									
								},
								fail: err => {
									uni.showToast({
										icon: 'none',
										title: "订单信息发送失败，请重试！"
									});
								}
							})
						},
						fail: err => {
							uni.showToast({
								icon: 'none',
								title: "获取token失败，请重试！"
							});
						}
					}) -->