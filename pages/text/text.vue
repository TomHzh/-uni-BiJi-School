<template>
	<view class="box">
		<view class="date">
			<i class="iconfont" data-method="undo" @tap="edit">
				<uni-icons type="undo-filled" size="25" color="#5e6d82"></uni-icons>
			</i>
			<i class="iconfont" data-method="redo" @tap="edit">
				<uni-icons type="redo-filled" size="25" color="#5e6d82"></uni-icons>
			</i>
			<i @tap="save">
				<uni-icons type="checkmarkempty" size="28"></uni-icons>
			</i>
		</view>

		<view class="editor_toolbox">
			<i class="iconfont icon-img" data-method="insertImg" @tap="edit" />
			<i class="iconfont icon-video" data-method="insertVideo" @tap="edit" />
			<uni-icons type="mic" size="30" @click="audiodd"></uni-icons>
			<i class="iconfont icon-link" data-method="insertLink" @tap="edit" />
			<i class="iconfont icon-text" data-method="insertText" @tap="edit" />
			<i class="iconfont icon-clear" @tap="clear" />

		</view>
		<view style="margin-top: 80rpx;" @click="da">
			<u--textarea v-model="value1" focus autoHeight border="bottom" confirmType="done" placeholder="请输入标题">
			</u--textarea>
			<!-- 小标题 -->
			<u--input class="inputXi" placeholder="请输入副标题" border="none" v-model="value" prefixIcon="pushpin">
			</u--input>

			<mp-html ref="article" container-style="padding:20px" :content="content"
				domain="https://mp-html.oss-cn-hangzhou.aliyuncs.com" :editable="editable" placeholder='开始书写'
				@remove="remove" />
		</view>
		<!-- 音频 -->
		<!-- <uni-file-picker 
			v-model="list" 
			fileMediatype="all" 
			mode="grid" 
			file-extname="mp3"
			@select="select" 
			@progress="progress" 
			@success="success" 
			@fail="fail" 
		/> -->

		<block v-if="modal">
			<view class="mask" />
			<view class="modal">
				<view class="modal_title">{{modal.title}}</view>
				<input class="modal_input" :value="modal.value" maxlength="-1" auto-focus @input="modalInput" />
				<view class="modal_foot">
					<view class="modal_button" @tap="modalCancel">取消</view>
					<view class="modal_button" style="color:#576b95;border-left:1px solid rgba(0,0,0,.1)"
						@tap="modalConfirm">确定</view>
				</view>
			</view>
		</block>
		<!-- 提示 -->
		<u-toast class="toast" ref="uToast"></u-toast>
	</view>

</template>

<script>
	import mpHtml from '@/components/mp-html/mp-html'
	const db = uniCloud.database()
	// 上传图片方法
	function upload(src, type) {
		return new Promise((resolve, reject) => {
			console.log('上传', type === 'img' ? '图片' : '视频', '：', src)
			// resolve(src)
			console.log('hhh', src);
			console.log(src.name, typeof(src.name));

			let name
			if (src.name) {
				name = src.name
			} else {
				let jpg = src.path
				// 获取后缀
				var suffix = jpg.substring(jpg.lastIndexOf(".") + 1)
				name = Date.now() + '.' + suffix
			}
			uniCloud.uploadFile({
					filePath: src.path,
					cloudPath: name
				}).then(
					(res) => {
						resolve(res.fileID)
					})
				.catch(err => {
					console.log('错', err);
				})
		})
	}
	// 删除图片方法
	function remove(src) {
		console.log('删除图片：', src)
		// 实际使用时，删除线上资源
	}
	export default {
		data() {
			return {
				list:[],
				value1: '',
				value: '',
				content: '',
				modal: null,
				editable: true,
				// 判断底部状态栏
				i_show: false,
				// 富文本类容
				editContent: '',
				// 封面大图
				picurls: '',
				// 判断修改还是增加的判断id
				id: '',
				// 音乐本地链接
				// 音频所需
			}
		},
		components: {
			mpHtml
		},
		onLoad(e) {
			this.getDataXiang(e.id);
			this.id = e.id;
			console.log('swa', this.id);

		},
		onReady() {
			/**
			 * @description 设置获取链接的方法
			 * @param {String} type 链接的类型（img/video/audio/link）
			 * @param {String} value 修改链接时，这里会传入旧值
			 * @returns {Promise} 返回线上地址
			 *   type 为音视频时可以返回一个数组作为源地址
			 *   type 为 audio 时，可以返回一个 object，包含 src、name、author、poster 等字段
			 */
			this.$refs.article.getSrc = (type, value) => {
				return new Promise((resolve, reject) => {
					if (type === 'img' || type === 'video') {
						uni.showActionSheet({
							itemList: ['本地选取'],
							success: res => {
								if (res.tapIndex === 0) {
									// 本地选取
									if (type === 'img') {
										uni.chooseImage({
											count: value === undefined ? 9 :
											1, // 2.2.0 版本起插入图片时支持多张（修改图片链接时仅限一张）
											success: res => {
												// console.log('awq',res);
												// #ifdef MP-WEIXIN
												if (res.tempFilePaths.length == 1 && wx
													.editImage) {
													// 单张图片时进行编辑
													wx.editImage({
														src: res.tempFilePaths[
															0],
														complete: res2 => {
															uni.showLoading({
																title: '上传中'
															})
															upload(res2
																	.tempFilePath ||
																	res
																	.tempFilePaths[
																		0],
																	type)
																.then(
																	res => {
																		uni.hideLoading()
																		resolve
																			(
																				res)
																	})
														}
													})
												} else {
													// #endif
													uni.showLoading({
														title: '上传中'
													});
													(async () => {
														const arr = []
														for (let item of res
																.tempFiles) {
															// 依次上传
															// console.log('sde',item);
															const src =
																await upload(
																	item, type)
															// console.log('src',src);
															arr.push(src);

														}
														return arr
													})().then(res => {
														// console.log('hello',res);
														uni.hideLoading()
														// console.log('jie');
														resolve(res)
													})
													// #ifdef MP-WEIXIN
												}
												// #endif
											},
											fail: reject
										})
									} else {
										uni.chooseVideo({
											success: res => {
												let tempFile = {
													name: res.name,
													path: res.tempFilePath
												}
												// console.log('asd',res);
												uni.showLoading({
													title: '上传中'
												});
												upload(tempFile, type).then(res => {
													uni.hideLoading()
													resolve(res)
												})
											},
											fail: reject
										})
									}
								} else {
									// 远程链接
									this.callback = {
										resolve,
										reject
									}
									this.$set(this, 'modal', {
										title: (type === 'img' ? '图片' : '视频') + '链接',
										value
									})
								}
							}
						})
					} else {
						this.callback = {
							resolve,
							reject
						}
						let title
						if (type === 'audio') {
							title = '音频链接'
						} else if (type === 'link') {
							title = '链接地址'
						}
						this.$set(this, 'modal', {
							title,
							value
						})
					}
				})
			}
		},

		methods: {
			// 上传音频
			// 获取上传状态
						select(e){
							console.log('选择文件：',e)
						},
						// 获取上传进度
						progress(e){
							console.log('上传进度：',e)
						},
						
						// 上传成功
						success(e){
							console.log('上传成功',e)
							this.list=e.tempFiles
						},
						
						// 上传失败
						fail(e){
							console.log('上传失败：',e)
						},
			audiodd() {
				console.log('ghg');
			},
			// 查询对应数据
			getDataXiang(id) {
				db.collection("note").doc(id).get().then(
					(res) => {
						this.value1 = res.result.data[0].title;
						this.value = res.result.data[0].subhead;
						this.content = res.result.data[0].content;
					})
			},
			//打开底部状态栏
			 da() {
				this.i_show = true
				this.editable = true
				// await db.collection("note").doc(this.id).get().then(
				// (res)=>{
				// 	this.value1=res.result.data[0].title;
				// 	this.value=res.result.data[0].subhead;
				// 	this.content=res.result.data[0].content;
				// 	this.i_show=true
				// 	this.editable=true
				// })

			},
			btn() {
				setTimeout(() => {
					var ctx = this.$refs.article
					var cover = ctx.imgList;
					this.picurls = cover[0]
				 console.log(this.picurls, typeof(this.picurls));

					let data = {
						title: this.value1,
						subhead: this.value,
						content: this.$refs.article.getContent(),
						picurls: this.picurls,
						posttime: Date.now()
				 }
					if (this.id) {
						db.collection("note").doc(this.id).update({
								...data
							}).then(
								(res) => {
									// console.log(res);
									uni.showToast({
										title: '保存成功'
									})
								})
						 .catch((err) => {
								uni.showToast({
									title: '保存失败',
									icon: 'error'
								})
							})
					} else {
						db.collection("note").add({
								...data
							}).then(
								(res) => {
									// console.log(res);
									uni.showToast({
										title: '保存成功'
									})
								})
							.catch(
								(err) => {
									console.log(err);
									this.$refs.uToast.show({
										type: 'error',
										duration: 1500,
										icon: 'close-circle-fill',
										message: err + '',
									})
								})
					}


				}, 100)


				// this.editable=true
			},
			// 删除图片/视频/音频标签事件
			remove(e) {
				// 删除线上资源
				remove(e.src)
			},
			// 处理模态框
			modalInput(e) {
				this.value = e.detail.value
			},
			modalConfirm() {
				this.callback.resolve(this.value || this.modal.value || '')
				this.$set(this, 'modal', null)
			},
			modalCancel() {
				this.callback.reject()
				this.$set(this, 'modal', null)
			},
			// 调用编辑器接口
			edit(e) {
				this.$refs.article[e.currentTarget.dataset.method]()
			},
			// 清空编辑器内容
			clear() {
				uni.showModal({
					title: '确认',
					content: '确定清空内容吗？',
					success: res => {
						if (res.confirm)
							this.$refs.article.clear()
						this.value1 = ''
					}
				})
			},
			// 保存编辑器内容
			save() {
				console.log('sd',this.list);
				console.log('s', this.audiolist);
				setTimeout(() => {
					var content = this.$refs.article.getContent()
					uni.showModal({
						title: '保存',
						content:'保存以上数据',
						confirmText: '完成',
						success: res => {
							if (res.confirm) {
								this.btn();
								this.editContent=this.$refs.article.getContent()
								this.editable=false
								this.i_show = false
							}
						}
					})
				}, 50)
			}
		}
	}
</script>

<style scoped>
	.box {
		margin-top: 150rpx;
	}

	/* 小标题 */
	.inputXi {
		margin-top: 20rpx
	}

	/* 提示样式 */
	/deep/.u-icon__icon {
		font-size: 20px !important;
	}

	/* 去除标题底色 */
	.u-textarea {
		background-color: transparent;
		font-weight: 700;
	}

	.uni-textarea-compute,
	.uni-textarea-line,
	.uni-textarea-placeholder,
	.uni-textarea-textarea {
		font-size: 16px;
		line-height: 16px;
	}

	>>>.uni-textarea-textarea {
		color: #000;

	}

	/* 保存顶部样式 */
	.date {
		position: absolute;
		right: 0;
		margin-top: -60rpx;
	}

	.date i {
		padding-right: 30rpx;
	}

	.editor_toolbox {
		position: fixed;
		bottom: 0;
		width: 100%;
		z-index: 999;
		background-color: #f6f6f8;
		border-bottom: 30rpx solid #f6f6f8;
		display: flex;
		padding: 5px;
		align-items: center;
		box-sizing: border-box;
		line-height: 1.6;
	}

	@font-face {
		font-family: "iconfont";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,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') format('woff2');
	}

	.iconfont {
		flex: 1;
		text-align: center;
		font-family: "iconfont" !important;
		font-size: 22px;
		font-style: normal;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	.iconfont:active {
		color: red;
	}

	.icon-undo:before {
		content: "\e607";
	}

	.icon-redo:before {
		content: "\e606";
	}

	.icon-img:before {
		content: "\e6e2";
	}

	.icon-video:before {
		content: "\e798";
	}

	.icon-link:before {
		content: "\e60d";
	}

	.icon-text:before {
		content: "\e6ce";
	}

	.icon-clear:before {
		content: "\e637";
	}

	.icon-save:before {
		content: "\e501";
	}

	/* 模态框 */
	.modal {
		position: fixed;
		top: 50%;
		left: 16px;
		right: 16px;
		background-color: #fff;
		border-radius: 12px;
		transform: translateY(-50%);
	}

	.modal_title {
		padding: 32px 24px 16px;
		font-size: 17px;
		font-weight: 700;
		text-align: center;
	}

	.modal_input {
		display: block;
		padding: 5px;
		margin: 0 24px 32px 24px;
		font-size: 14px;
		border: 1px solid #dfe2e5;
	}

	.modal_foot {
		display: flex;
		line-height: 56px;
		font-weight: 700;
		border-top: 1px solid rgba(0, 0, 0, .1);
	}

	.modal_button {
		flex: 1;
		text-align: center;
	}

	/* 蒙版 */
	.mask {
		position: fixed;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		background-color: black;
		opacity: 0.5;
	}

	/* 音频样式 */
	.boxmic {
		width: 90vw;
		margin: 0 auto;
		padding-top: 10vw;
		background-color: #fff;
	}

	.contmic {
		height: 20vw;
		width: 90vw;
		margin: 0 auto;
		margin-top: -10vw;
		background-color: rgba(0, 0, 0, .5);
		/* filter: drop-shadow(
		0px 3rpx 5rpx rgba(85, 170, 255, 0.6) 
		); */
		filter: drop-shadow(0px 3rpx 9rpx rgba(0, 0, 0, 1));
		border-radius: 15rpx;
	}

	.tumic {
		height: 20vw;
		width: 20vw;
		position: absolute;
		border-radius: 15rpx;
	}

	.icon-playmic {
		width: 5vw;
		height: 5vw;
		z-index: 999;
		margin-left: 7vw;
		margin-top: 7vw;
	}

	.rightmic {
		margin-left: 23vw;
		margin-top: -12vw;
		display: flex;
		justify-content: space-between;
	}

	.rightmic .namemic {
		line-height: 70rpx;
		color: #fff;
	}

	.rightmic .authormic {
		color: #55ff00;
	}

	.rightmic .timemic {
		line-height: 70rpx;
		margin-right: 5vw;
		color: #cfcfcf;
	}
</style>
