<template>
	<view class="container">
		<view class="row b-b">
			<text class="tit">商家名称</text>
			<input disabled class="input" type="text" v-model="merchantName" placeholder-class="placeholder" />
		</view>
		<view class="row b-b">
			<text class="tit">联系人</text>
			<input class="input" type="text" maxlength="20" v-model="contactName" placeholder="不超过20个字符" placeholder-class="placeholder" />
		</view>
		<view class="row b-b">
			<text class="tit">手机号码</text>
			<input class="input" type="text" maxlength="20" v-model="mobileNo" placeholder="不超过20个字符" placeholder-class="placeholder" />
		</view>
		<view class="row b-b">
			<text class="tit">商家地址</text>
			<input @click="map" class="input" type="text" v-model="merchantAddress" placeholder="不超过20个字符" placeholder-class="placeholder" />
		</view>
		<button class="add-btn" @click="save">保存</button>
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
				merchantName: '',
				contactName: '',
				mobileNo: '',
				merchantAddress: '',
				longitude: '',
				latitude: ''
			};
		},
		computed: {
			...mapState(['hasLogin', 'merchantInfo'])
		},
		methods: {
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
			//保存昵称
			save() {
				if (!this.contactName) {
					this.$api.msg('请输入联系人');
					return;
				} else if (!this.$api.util.validateMobileNo(this.mobileNo)) {
					this.$api.msg('手机号码格式错误');
					return;
				} else if (!this.merchantAddress) {
					this.$api.msg('请选择商家地址');
					return;
				}
				this.$api.request.editInfo({
					actionType:'MODIFY',
					merchantUuid: this.merchantInfo.merchantUuid,
					merchantName: this.merchantName,
					mobileNo: this.mobileNo,
					merchantAddress: this.merchantAddress,
					contactName: this.contactName,
					longitude: this.longitude,
					latitude: this.latitude
				}, res => {
					if (res.body.status.statusCode === '0') {
						this.$api.msg('信息已保存');
						setTimeout(function() {
							uni.switchTab({
								url: '/pages/user/user'
							})
						}, 1000);
					} else {
						this.$api.msg(res.body.status.errorDesc);
					}
				});
			},

		},
		mounted() {
			this.merchantName = this.merchantInfo.merchantName,
			this.contactName = this.merchantInfo.contactName,
			this.mobileNo = this.merchantInfo.mobileNo,
			this.merchantAddress =this.merchantInfo.merchantAddress,
			this.longitude = this.merchantInfo.longitude,
			this.latitude = this.merchantInfo.latitude
		}
	}
</script>

<style lang='scss'>
	page {
		background: $page-color-base;
	}

	.row {
		display: flex;
		align-items: center;
		position: relative;
		padding: 0 30upx;
		height: 110upx;
		background: #fff;

		.tit {
			flex-shrink: 0;
			width: 160upx;
			font-size: 30upx;
			color: $font-color-dark;
		}

		.input {
			flex: 1;
			font-size: 30upx;
			color: $font-color-dark;
		}

		.icon-shouhuodizhi {
			font-size: 36upx;
			color: $font-color-light;
		}
	}

	.add-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 690upx;
		height: 80upx;
		margin: 60upx auto;
		font-size: $font-lg;
		color: #fff;
		background-color: $base-color;
		border-radius: 10upx;
		box-shadow: 1px 2px 5px rgba(219, 63, 96, 0.4);
	}
</style>
