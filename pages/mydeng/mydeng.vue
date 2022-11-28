<template>
	<view class="boxd">
		<view class="box">
		</view>
		<view class="top">
			<view class="img">
		<!-- 		<image
					src="/static/img/tou.svg" /> -->
				<uni-id-pages-avatar class="toimg"  width="260rpx" height="260rpx"></uni-id-pages-avatar>	
			</view>
			<view class="text" @click="login">
				<h2>{{deng}}</h2>
			</view>
		</view>

		<view class="cent">
			<view class="nei">
				<uni-list :border="false">
					<uni-list-item v-for="item in extraIcon" :title="item.tie.title" showArrow :border="false" :show-extra-icon="true"
						:extra-icon="item.icon" 
						thumb-size="base"
						 clickable
						@click="diaoFn(item.click)"
						:rightText="item.rightText"
						 />
				</uni-list>
				<uni-popup ref="dialog" type="dialog">
					<uni-popup-dialog mode="input" :value="userInfo.nickname" @confirm="setNickname" title="设置昵称"
						placeholder="请输入要设置的昵称">
					</uni-popup-dialog>
				</uni-popup>
				<uni-id-pages-bind-mobile ref="bind-mobile-by-sms" @success="bindMobileSuccess"></uni-id-pages-bind-mobile>
			</view>
		</view>
	</view>

</template>

<script>
	const db = uniCloud.database();
	const usersTable = db.collection('uni-id-users')
	const uniIdCo = uniCloud.importObject("uni-id-co")
	import {
		store,
		mutations
	} from '@/uni_modules/uni-id-pages/common/store.js'
	export default {
		computed: {
			userInfo() {
				this.extraIcon[1].rightText =store.userInfo.nickname || '未设置';
				return store.userInfo
			},
			deng(){
			return this.userInfo.nickname ? this.userInfo.nickname : this.userInfo.username ? this.userInfo.username : '登录'
				
			}
		},
		data() {
			return {
				univerifyStyle: {
					authButton: {
						"title": "本机号码一键绑定", // 授权按钮文案
					},
					otherLoginButton: {
						"title": "其他号码绑定",
					}
				},
				// userInfo: {
				// 	mobile:'',
				// 	nickname:''
				// },
				hasPwd: false,
				name:"登录",
				baseFormData: {
									name: '',
									age: '',
									introduction: '',
									sex: 2,
									hobby: [5],
									datetimesingle: 1627529992399
								},
					extraIcon: [
						{
							tie:{title:'我的笔记'},
							icon:{color: '#d90003',size: '22',type: 'pyq'},
							click:'ontoShow',
						},
						{
							tie:{title:'昵称'},
							icon:{color: '#d90003',size: '22',type: 'map-filled'},
							click:'setNickname',
							rightText:''
						},
						// {
						// 	tie:{title:'手机号'},
						// 	icon:{color: '#d90003',size: '22',type: 'phone-filled'},
						// 	click:'bindMobile',
						// 	rightText:''
						// },
						{
							tie:{title:'修改密码'},
							icon:{color: '#d90003',size: '22',type: 'locked-filled'},
							click:'changePassword'
						},
						{
							tie:{title:'注销账号'},
							icon:{color: '#d90003',size: '22',type: 'trash-filled'},
							click:'deactivate'
						},
						{
							tie:{title:'退出登录'},
							icon:{color: '#d90003',size: '22',type: 'redo-filled'},
							click:'logout'
						},
				],
				

			}
		},
		async onload(){
			let res = await uniIdCo.getAccountInfo()
			this.hasPwd = res.isPasswordSet
			
			
		},
		methods: {
			// 循环调用方法
			diaoFn(method){
				console.log(method);
				 this[method]()
			},
			// 跳到笔记页面
			ontoShow(){
				uni.switchTab({
					url:"/pages/index/index"
				})
			},
			// 手机号
			// bindMobile(){
			// 	console.log("ff");
			// 	//...去用验证码绑定
			// 	this.bindMobileBySmsCode()

			// },
			// univerify() {
			// 	uni.login({
			// 		"provider": 'univerify',
			// 		"univerifyStyle": this.univerifyStyle,
			// 		success: async e => {
			// 			console.log(e.authResult);
			// 			uniIdCo.bindMobileByUniverify(e.authResult).then(res => {
			// 				console.log(res);
			// 				mutations.updateUserInfo()
			// 			}).catch(e => {
			// 				console.log(e);
			// 			}).finally(e => {
			// 				console.log(e);
			// 				uni.closeAuthView()
			// 			})
			// 		},
			// 		fail: (err) => {
			// 			console.log(err);
			// 			if (err.code == '30002' || err.code == '30001') {
			// 				this.bindMobileBySmsCode()
			// 			}
			// 		}
			// 	})
			// },
			// // 绑定手机号页面
			// bindMobileBySmsCode(){
			// 	uni.navigateTo({
			// 		url: '/uni_modules/uni-id-pages/pages/userinfo/bind-mobile/bind-mobile'
			// 	})
			// },
			// 修改密码
			changePassword() {
				uni.navigateTo({
					url: '/uni_modules/uni-id-pages/pages/userinfo/change_pwd/change_pwd',
					complete: (e) => {
						console.log(e);
					}
				})
			},
			// 登录
			login() {
				
				if(uni.getStorageSync('uni_id_token')){
					console.log('dd');
				}else{
					console.log('hello');
					uni.navigateTo({
						url: '/uni_modules/uni-id-pages/pages/login/login-withpwd',
						complete: (e) => {
							console.log(e);
						}
					})
				}
				
			},
			// 退出登录
			logout() {
				mutations.logout()
			},
			// 注销账号
			deactivate() {
				uni.navigateTo({
					url: "/uni_modules/uni-id-pages/pages/userinfo/deactivate/deactivate"
				})
			},
			// 调用检测更新
			bindMobileSuccess() {
				mutations.updateUserInfo()
			},
			// 昵称
			setNickname(nickname) {
				console.log(this.userInfo);
				if (nickname) {
					mutations.updateUserInfo({
						nickname
					})
					this.$refs.dialog.close()
				} else {
					this.$refs.dialog.open()
				}
			},
			async bindThirdAccount(provider) {
				const uniIdCo = uniCloud.importObject("uni-id-co")
				const bindField = {
					weixin: 'wx_openid',
					alipay: 'ali_openid',
					apple: 'apple_openid',
					qq: 'qq_openid'
				} [provider.toLowerCase()]
			
				if (this.userInfo[bindField]) {
					await uniIdCo['unbind' + provider]()
					await mutations.updateUserInfo()
				} else {
					uni.login({
						provider: provider.toLowerCase(),
						onlyAuthorize: true,
						success: async e => {
							const res = await uniIdCo['bind' + provider]({
								code: e.code
							})
							if (res.errCode) {
								uni.showToast({
									title: res.errMsg || '绑定失败'
								})
							}
							await mutations.updateUserInfo()
						},
						fail: async (err) => {
							console.log(err);
							uni.hideLoading()
						}
					})
				}
			
			}
		}
	}
</script>

<style>
	.exampled{
		height: 120vw;
		background-color: #fff;
		border-radius: 5vw;
	}
	.example {
		width: 95vw;
		height: 100vw;
		background-color: #fff;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 5vw;
	}
	.box {
		height: 80vw;
		background-image: url("../../static/img/dengbei4.jpg");
		background-position: center;
		background-size: cover;
		border-radius: 0 0 15vw 15vw;
		/* 	background-attachment:fixed; */
	}

	.boxd .top {
		height: 30vw;
		width: 96vw;
		margin: -60vw auto;
		display: flex;
		align-items: center;
		/* border-radius: 6vw; */
		/* background-image: linear-gradient(to top left ,#312891 , #fa7bfa 80%); */
		background-color: transparent;
		/* box-shadow: 0 0 30rpx 15rpx rgba(0, 0, 0, .3); */
		/* border: 3rpx solid #000; */
	}

	.top .img {
		margin: 0 6vw;
	}

	.top .toimg {
		display: block;
		width: 16vw !important;
		height: 16vw !important;
		border-radius: 50%;
		border: 6rpx solid #fff;
		display: flex;
		align-items: center;
		justify-content: center;
		/* box-shadow: 0 0 30rpx 10rpx rgba(255, 255, 255, .5) ; */
	}

	.top .text {
		color: #fff;
		line-height: 8vw;
	}

	.top .text h2 {
		font-weight: 400;
		letter-spacing: 8rpx;
	}

	.boxd .cent {
		height: 100vw;
		width: 95vw;
		background-color: #fff;
		margin: 65vw auto;
		border-radius: 6vw;
		box-shadow: 0 0rpx 30rpx 5rpx rgba(0, 0, 0, .3);

	}

	.uni-list {
		border-radius: 10vw;
		padding-top: 6vw;
	}

	.uni-list-item {
		width: 80vw;
		margin: 2vw auto;
	}
	/* .bom{
		height: 13vw;
		width: 80vw;
		margin: 0 auto;
		background-color: #9b47ec;
		background-image: linear-gradient(10deg , #fe7ffe,#393939);
		border-radius: 10vw;
		margin-top: -30vw;
		margin-bottom: 5vw;
		text-align: center;
		line-height: 13vw;
		box-shadow: 0 8rpx 20rpx 5rpx rgba(0, 0, 0, .3);
		border: 5rpx solid #fff;
	}
	.bom span{
		color: #fff;
		font-size: 40rpx;
		letter-spacing: 15rpx;
	} */
</style>
