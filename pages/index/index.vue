<template>
	<view class="mar">
		<view class="imga">
			<!-- <image src="../../static/img/dengbei4.jpg" mode="" @click="geng"></image> -->
			<image :src="fengSrc" mode="aspectFill" @click="geng"></image>
		</view>

		<view class="biji">
			<!-- 头部 -->
			<view class="top">
				<view class="shang">
					<h1>全部笔记</h1>
					<u-icon name="arrow-down-fill"></u-icon>
				</view>
				<p><span>{{dataList.length}}条笔记</span></p>
			</view>

			<!-- 搜索框 -->
			<view class="sou">
				<u-search bgColor="#fff" :clearabled="true" :showAction="false" shape="round" :animation="true"
					searchIconSize="50" margin="50rpx 0" height="90rpx" placeholder="搜索笔记" v-model="keyword" @change="change"></u-search>
			</view>

			<!-- 内容卡片 -->

			<view class="box">
				<u-empty mode="data" icon="http://cdn.uviewui.com/uview/empty/data.png" textSize=30 width=400 height=400
					:show='showEmpty' />
				<u-empty mode="permission" icon="http://cdn.uviewui.com/uview/empty/permission.png" textSize=30
					width=400 height=400 text="无权限请登录" :show='showEmptypermission' />

				<view class="pan" v-if="dataList">
					<view :class="index%2==0 ? 'xun':'xun2'" v-for="(item,index) in dataList" :key="item._id"
						@longpress="deletes(item._id)" @click="notaBian(item._id)">
						<view class="ka">
							<view class="left">
								<view class="text">
									<h2>{{item.title}}</h2>
									<p><span>{{item.subhead==null || item.subhead==undefined || item.subhead=="" ? '无副标题' : item.subhead}}</span>
									</p>
									<p><span>
											<uni-dateformat :date="item.posttime" format="yyyy/MM/dd hh:mm:ss"
												:threshold="[60000, 3600000]"></uni-dateformat>
										</span></p>
								</view>
							</view>



							<view class="right" v-if="item.picurls">
								<image :src="item.picurls" mode=""></image>
							</view>
						</view>
						<!-- 模态框 -->
						<u-modal :show="showDatele" @confirm="confirmDatele(item._id)" @cancel="cancelDatele"
							ref="uModal" title="是否确认删除此条记录" showCancelButton confirmColor="#f56c6c"
							cancelColor="#3c9cff" :asyncClose="true">
						</u-modal>
					</view>
				</view>

			</view>
		</view>
		<!-- 上传提示 -->
		<view>
			<u-toast ref="uToast"></u-toast>
		</view>
		<view class="tian">
			<uni-fab ref="fab" :pattern="pattern" horizontal="right" vertical="bottom" @fabClick="fabClick('text')" />
		</view>
		<u-modal :show="show" :showCancelButton="true" title="是否更换封面" :zoom="false" @confirm='que' @cancel="guan">
		</u-modal>
		<!-- 弹出换背景 -->
		<u-popup closeable overlay bgColor="#fff" safeAreaInsetBottom safeAreaInsetTop :customStyle="popClass"
			:show="show1" :round="60" mode="bottom" @close="close" @open="open">
			<view>

				<uni-file-picker ref="files" v-model="imageValue" fileMediatype="image" mode="grid" :auto-upload="false"
					limit=1 @select="select" @progress="progress" @success="success" @delete="deleted" @fail="fail"
					:image-styles="imageStyles" />
				<uni-section v-if="!btn" titleColor="#f00" titleFontSize="16px" title="请上传封面图片" type="line">
				</uni-section>
				<u--text v-if="btn" v-for="item in nameList" :key="item.uuid" type="info" :text="item.name" size="32"
					margin="20rpx 0"></u--text>
				<u-button v-if="btn && !disabled" @click="btnhui" size="large" style="margin: 0 0 50rpx 0;"
					type="primary" text="确定"></u-button>
				<u-button v-if="btn && disabled" type="primary" loading loadingMode="circle" size="large" disabled
					:loadingSize='100' loadingText=上传中></u-button>
			</view>
		</u-popup>
	</view>
</template>

<script>
	const db = uniCloud.database()
	export default {
		data() {
			return {
				// 类容为空组件判断
				showEmpty: false,
				showEmptypermission: false,
				show: false,
				show1: false,
				shows: true,
				// 删除时间的的判断模态框
				showDatele: false,
				btn: false, //判断是否显示对应的按钮和文件名
				disabled: false, //判断上传完毕没有
				nameList: null, //上传图片的数据
				fengSrc: "", //封面
				imageStyles: {
					width: 240,
					height: 240,
				},
				imageValue: [],
				keyword: '',//搜索框值
				pattern: {
					buttonColor: "#ffbb02"
					// backgroundColor:"#000"
				},
				popClass: {
					width: '100vw',
					display: 'flex',
					'align-items': 'center',
					'justify-content': 'center',
				},
				successd: {
					type: 'success',
					message: "上传成功",
					iconUrl: 'https://cdn.uviewui.com/uview/demo/toast/success.png'
				},
				err: {
					type: 'error',
					icon: false,
					message: "上传失败",
					iconUrl: 'https://cdn.uviewui.com/uview/demo/toast/error.png'
				},
				// 数据数组
				dataList: [],
				// 获取缓存里面的用户信息
				data: ''
			}
		},
		async onLoad() {
			await this.getData()
			this.getuserStor()
			this.getCover()
			this.empty()
		},
		onShow() {
			this.getData()
			this.getuserStor()
			this.getCover()
		},
		methods: {
			// 搜索框后
			change(value){
				console.log(value);
				if(value){
					for (let item of this.dataList) {
						let res=this.dataList.filter((p)=>{
							return p.title.toUpperCase().indexOf(value.toUpperCase()) !== -1
						} )
						this.dataList=res
				}
				}else{
					this.getData()
				}
				
			},
			// 判断用户登录还是无数据
			empty() {
				const auth = uniCloud.getCurrentUserInfo()
				this.tokDate = auth.tokenExpired
				if (uni.getStorageSync('uni_id_token')) {
					// 判断token过期
					if (auth.tokenExpired - Date.now() > 0) {
						if (this.dataList.length <= 0) {
							this.showEmpty = true
						} else {
							this.showEmpty = false;
							this.showEmptypermission = false;

						}
					} else {
						this.dataList = ''
						this.showEmptypermission = true
					}

				} else {
					this.dataList = ''
					this.showEmptypermission = true
				}
			},
			// 获取缓存里面的用户信息
			getuserStor() {
				this.data = uni.getStorageSync('uni-id-pages-userInfo')
			},
			//删除对应笔记
			deletes() {
				this.showDatele = true
			},
			// 确认
			confirmDatele(id) {
				console.log(id);
				db.collection("note").doc(id).remove().then(
					(res) => {
						this.showDatele = false
						uni.showToast({
							title: '删除成功',
							duration: 1500
						});
					})
			},
			// 取消
			cancelDatele() {
				this.showDatele = false
			},
			// 跳转详情页面
			notaBian(e) {
				uni.navigateTo({
					url: '/pages/text/text?id=' + e
				})
			},
			// 查询数据
			getData() {
				let artTemp = db.collection("note").where('user_id==$cloudEnv_uid').getTemp();
				let userTemp = db.collection("uni-id-users").field('_id').getTemp();
				db.collection(artTemp, userTemp).field('title,posttime,subhead,picurls').orderBy("posttime", 'desc').get()
					.then(
						(res) => {
							console.log('data',res.result.data);
							this.dataList = res.result.data;
							this.empty()
						})
					.catch((err) => {
						console.log('获取错', err);
					})
			},
			fabClick(e) {
				const url = '/pages/' + e + '/' + e
				uni.navigateTo({
					url: url,
				})

			},
			open() {
				// console.log('open');
			},
			close() {
				this.show1 = false
				// console.log('close');
			},

			geng() {
				this.show = true
			},
			que() {
				this.show = false;
				this.show1 = true
			},
			guan() {
				this.show = false
			},
			// 上传确认按钮
			btnhui() {
				this.$refs.files.upload()
				this.disabled = true
			},
			//上传封面
			// 选择文件触发
			select(e) {
				console.log('22', e);
				this.nameList = e.tempFiles
				this.btn = true
			},
			// 移除列表时触发
			deleted() {
				this.btn = false
				this.disabled = false
			},
			//上传文件时触发
			progress() {},
			// 上传成功
			async success(res) {
				console.log('11', res);
				this.disabled = false
				this.show1 = false
				const params = this.successd
				this.showToast(params)
				this.imageValue = null
				this.btn = false
				let cover = res.tempFilePaths[0]
				await db.collection("uni-id-users").doc(this.data._id).update({
					cover
				}).then(
					(res) => {
						// console.log("该封面", res);
						this.getCover()
					})
			},
			// 上传成功后查询封面
			getCover() {
				db.collection("uni-id-users").doc(this.data._id).field('cover').get().then(
					(res) => {
						// console.log("封面", res);
						this.fengSrc = res.result.data[0].cover

					})
			},
			//上传失败
			fail() {
				this.disabled = false
				this.show1 = false
				const parerr = this.err
				this.showToast(parerr)
				this.imageValue = null
				this.btn = false
			},

			showToast(params) {
				console.log(params);
				this.$refs.uToast.show({
					...params
				})
			}

		}
	}
</script>

<style lang="less">
	
	page {
		margin: 0;
		padding: 0;
	}

	// /deep/.u-upload__button{
	// 	margin: 0;
	// }
	// /deep/.u-upload{
	// 	margin: 100rpx 0 30rpx 0;
	// }

	/deep/.u-popup__content__close--top-right .u-icon__icon>span {
		font-size: 40rpx;
	}

	// 背景上传的样式
	.uni-file-picker {
		margin: 60rpx 0 30rpx 0;
	}

	.uni-section {
		margin-bottom: 20rpx;
		background-color: transparent;
	}

	/deep/.uni-fab__circle--rightBottom {
		right: calc(65rpx + var(--window-right));
	}

	// 按钮上的加载图标
	/deep/.u-loading-icon__spinner {
		width: 40rpx !important;
		height: 40rpx !important;
	}

	.imga {
		width: 100vw;
		position: absolute;

		// height: 100vw;
		image {
			display: block;
			width: 100vw;
			height: 400rpx;
		}
	}

	.mar {
		// margin-top: 100rpx;
	}

	.biji {
		border-radius: 60rpx 60rpx 0 0;
		background-color: #f4f4f4;
		position: relative;
		top: 140px;
	}

	.mar .top {
		width: 90vw;
		margin: 0 auto;
	}

	.mar .top .shang {
		display: flex;
		width: 300rpx;
		justify-content: space-between;
		line-height: 90rpx;
	}

	.mar .sou {
		width: 90vw;
		margin: 0 auto;
	}

	.box .pan .xun {
		width: 90vw;
		margin: 0 auto;
		background-color: #fff;
		border-radius: 30rpx;
		margin-bottom: 20rpx;
	}

	.box .pan .xun2 {
		width: 90vw;
		margin: 0 auto;
		background-color: #767a82;
		border-radius: 30rpx;
		margin-bottom: 20rpx;
	}

	.box .pan .xun .ka {
		width: 80vw;
		margin: 0 auto;
		display: flex;
		align-items: center;
		justify-content: space-between;


		.text {
			margin: 30rpx 0;
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			line-height: 50rpx;

			h2 {
				font-weight: 600;
			}

			span {
				color: #7e7e7e;
			}
		}


		.right {
			width: 130rpx;
			height: 130rpx;

			image {
				display: block;
				width: 130rpx;
				height: 130rpx;
			}
		}

	}

	.box .pan .xun2 .ka {
		width: 80vw;
		margin: 0 auto;
		display: flex;
		align-items: center;
		justify-content: space-between;


		.text {
			margin: 30rpx 0;
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			line-height: 50rpx;

			h2 {
				font-weight: 600;
				color: #fff;
			}

			span {
				color: #efefef;
			}
		}


		.right {
			width: 130rpx;
			height: 130rpx;

			image {
				display: block;
				width: 130rpx;
				height: 130rpx;
			}
		}

	}
</style>
