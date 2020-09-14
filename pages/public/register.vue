<template>
	<view class="container">
		<view class="left-bottom-sign"></view>
		<view class="back-btn yticon icon-zuojiantou-up" @click="navBack"></view>
		<view class="right-top-sign"></view>
		<!-- 设置白色背景防止软键盘把下部绝对定位元素顶上来盖住输入框等 -->
		<view class="wrapper">
			<view class="left-top-sign">REGISTER</view>
			<view class="welcome">
				{{applicationConfig.applicationName}}
			</view>
			<view class="welcome">
				欢迎商家入驻！
			</view>
			<view class="input-content">
				<view class="input-item">
					<text class="tit">商家名称</text>
					<input type="text" :value="merchantName" placeholder="请输入商家名称" maxlength="30" data-key="merchantName" @input="inputChange" />
				</view>
				<view class="input-item">
					<text class="tit">联系人</text>
					<input type="text" :value="contactName" placeholder="请输入联系人姓名" maxlength="11" data-key="contactName" @input="inputChange" />
				</view>
				<view class="input-item">
					<text class="tit">手机号码</text>
					<input type="number" :value="mobileNo" placeholder="请输入手机号码" maxlength="11" data-key="mobileNo" @input="inputChange" />
				</view>
				<view class="input-item">
					<text class="tit">所在城市</text>
					<view style="font-size: 15px;color:#303133" @click="showAddressRegion">
						{{province}}  {{city}}  {{district}}
					</view>
				</view>
				<view class="input">
					<w-picker
						mode="region"
						:defaultVal="defaultRegion"
						:hideArea="false"
						@confirm="onConfirm" 
						ref="region"
						:timeout="true"
					></w-picker>
				</view>
				<view class="input-item">
					<text class="tit">详细地址</text>
					<input disabled @click="map()" type="text" :value="merchantAddress" placeholder="点击选择商家地址" placeholder-class="input-empty" maxlength="20"
					 data-key="merchantAddress" @input="inputChange" />
				</view>
			</view>
			<!-- #ifdef MP-WEIXIN -->
			<button open-type="getUserInfo" class="confirm-btn" @getuserinfo="getuserinfo" withCredentials="true" :disabled="registering">注册</button>
			<!-- #endif -->
			
			<!-- #ifndef MP-WEIXIN -->
			<button class="confirm-btn" @click="toRegister" :disabled="registering">注册</button>
			<!-- #endif -->
		</view>
	</view>
</template>

<script>
	import {
		mapState,
		mapMutations
	} from 'vuex';

	export default {
		data() {
			return {
				province: '北京市',
				city: '市辖区',
				district: '东城区',
				defaultRegion:['北京市','市辖区','东城区'],
				merchantName: '',
				merchantAddress: '',
				latitude: null,
				longitude: null,
				contactName: '',
				mobileNo: '',
				registering: false,
				wechatUserInfo:{},
				to: '',
				suscribeMsgList:[]
			}
		},
		onLoad(options) {
			var to = options.to;
			if(to){
				this.to = unescape(to);
			}
			this.inquirySuscribeMsg();
		},
		computed: {
			...mapState(['applicationConfig'])
		},
		methods: {
			...mapMutations(['login']),
			showAddressRegion(){
				this.$refs['region'].show();	
			},
			onConfirm(val){
				this.province=val.checkArr[0];
				this.city=val.checkArr[1];
				this.district=val.checkArr[2];
			},
			map() {
				var that = this;
				uni.chooseLocation({
					success: function (res) {
						if(res.name){
							that.merchantAddress = res.address;
							that.latitude= res.latitude;
							that.longitude=res.longitude;
						}
					}
				});
			},
			inputChange(e) {
				const key = e.currentTarget.dataset.key;
				this[key] = e.detail.value;
			},
			navBack() {
				uni.navigateBack();
			},
			getuserinfo(e){
				let that = this;
				that.wechatUserInfo = e.detail.userInfo;
				uni.requestSubscribeMessage({
					// 订阅消息
					tmplIds: this.suscribeMsgList,
					success: function(res) {
						console.log('订阅消息订阅成功');
					}
				});
				uni.login({
					provider: 'weixin',
					success: function(res) {
						console.log('code: '+res.code);
						that.wechatUserInfo.code = res.code;
						that.toRegister();
					}
				});
			},
			//查询订阅消息
			inquirySuscribeMsg() {
				this.$api.request.wxSuscribeMsg({}, res => {
					if (res.body.status.statusCode === '0') {
						this.suscribeMsgList = res.body.data.suscribeMsgList;
					} else {
						console.log(res.body.status.errorDesc);
					}
				});
			},
			async toRegister() {
				this.registering = true;
				const {
					merchantName,
					mobileNo,
					contactName,
					province,
					city,
					district,
					merchantAddress,
					latitude,
					longitude
				} = this;
				var isFormValid = true;
				if (!merchantName) {
					this.$api.msg('请输入商家名称');
					isFormValid = false;
				} else if (!contactName) {
					this.$api.msg('请输入联系人');
					isFormValid = false;
				} else if (!this.$api.util.validateMobileNo(mobileNo)) {
					this.$api.msg('手机号码格式错误');
					isFormValid = false;
				} else if (!merchantAddress) {
					this.$api.msg('请选择商家地址');
					isFormValid = false;
				}
				if (!isFormValid) {
					this.registering = false;
					return;
				}
				let requestData = {
					merchantName: merchantName,
					mobileNo: mobileNo,
					contactName: contactName,
					province: province,
					city: city,
					district: district,
					merchantAddress: merchantAddress,
					latitude: latitude,
					longitude: longitude
				};
				this.$api.request.applyMerchant(requestData, regRes => {
					if (regRes.body.status.statusCode === '0') {
						uni.navigateTo({
							url: '/pages/public/registerSuccess'
						})
					} else {
						this.$api.msg(regRes.body.status.errorDesc);
						this.registering = false;
					}
				});
				// const result = await this.$api.json('userInfo');
				// if (result.status === 1) {
				// 	this.login(result.data);
				// 	uni.navigateBack();
				// } else {
				// 	this.$api.msg(result.msg);
				// 	this.registering = false;
				// }
			}
		}
	}
</script>

<style lang='scss'>
	page {
		background: #fff;
	}

	.container {
		padding-top: 40px;
		position: relative;
		width: 100vw;
		overflow: hidden;
		background: #fff;
	}

	.wrapper {
		position: relative;
		z-index: 90;
		background: #fff;
		padding-bottom: 40upx;
	}

	.back-btn {
		position: absolute;
		left: 40upx;
		z-index: 9999;
		padding-top: var(--status-bar-height);
		top: 40upx;
		font-size: 40upx;
		color: $font-color-dark;
	}

	.left-top-sign {
		font-size: 120upx;
		color: $page-color-base;
		position: relative;
		left: -16upx;
	}

	.right-top-sign {
		position: absolute;
		top: 80upx;
		right: -30upx;
		z-index: 95;

		&:before,
		&:after {
			display: block;
			content: "";
			width: 400upx;
			height: 80upx;
			background: #b4f3e2;
		}

		&:before {
			transform: rotate(50deg);
			border-radius: 0 50px 0 0;
		}

		&:after {
			position: absolute;
			right: -198upx;
			top: 0;
			transform: rotate(-50deg);
			border-radius: 50px 0 0 0;
			/* background: pink; */
		}
	}

	.left-bottom-sign {
		position: absolute;
		left: -270upx;
		bottom: -320upx;
		border: 100upx solid #d0d1fd;
		border-radius: 50%;
		padding: 180upx;
	}

	.welcome {
		position: relative;
		left: 50upx;
		top: -90upx;
		font-size: 46upx;
		color: #555;
		text-shadow: 1px 0px 1px rgba(0, 0, 0, .3);
	}

	.input-content {
		padding: 0 60upx;
	}

	.input-item {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		justify-content: center;
		padding: 0 30upx;
		background: $page-color-light;
		height: 120upx;
		border-radius: 4px;
		margin-bottom: 50upx;

		&:last-child {
			margin-bottom: 0;
		}

		.tit {
			height: 50upx;
			line-height: 56upx;
			font-size: $font-sm+2upx;
			color: $font-color-base;
		}

		input {
			height: 60upx;
			font-size: $font-base + 2upx;
			color: $font-color-dark;
			width: 100%;
		}
	}

	.confirm-btn {
		width: 630upx;
		height: 76upx;
		line-height: 76upx;
		border-radius: 50px;
		margin-top: 70upx;
		background: $uni-color-primary;
		color: #fff;
		font-size: $font-lg;

		&:after {
			border-radius: 100px;
		}
	}

	.forget-section {
		font-size: $font-sm+2upx;
		color: $font-color-spec;
		text-align: center;
		margin-top: 40upx;
	}

	.register-section {
		position: absolute;
		left: 0;
		bottom: 50upx;
		width: 100%;
		font-size: $font-sm+2upx;
		color: $font-color-base;
		text-align: center;

		text {
			color: $font-color-spec;
			margin-left: 10upx;
		}
	}
</style>
