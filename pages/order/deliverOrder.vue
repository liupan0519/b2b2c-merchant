<template>
	<view class="container">
		<!-- 快递配送 -->
		<view v-if="order.deliveryType=='1'">
			<view class="row">
				<text class="tit">快递公司</text>
				<picker class="input" @change="bindPickerChange" :value="index" :range="couriers">
										<view class="uni-input">{{couriers[index]}}</view>
									</picker>
				<text class="yticon icon-you"></text>
			</view>
			<view class="row">
				<text class="tit">快递单号</text>
				<input class="input" v-model="courierNo" placeholder="物流单号" placeholder-class="placeholder" />
			</view>
			<button class="add-btn" @click="save">确认</button>
		</view>
		<!-- 同城配送 -->
		<view v-if="order.deliveryType=='2'">
			<view class="row" v-if="shoppers.length>0">
				<text class="tit">配送员</text>
				<picker class="input" range-key="name" @change="shopperChange" :value="index" :range="shoppers">
					<view class="uni-input">{{shoppers[index].name}}</view>
				</picker>
				<text class="yticon icon-you"></text>
			</view>
			<view v-else>
				<view class="empty">
					您还没有没有配送员, 请先在PC端同城配送-配送人员中添加配送员
				</view>
			</view>
			<button  v-if="shoppers.length>0" class="add-btn" @click="save">确认</button>
			<button  v-if="shoppers.length==0" class="add-btn" @click="back">返回</button>
		</view>
		<!-- 门店自提 -->
		<view v-if="order.deliveryType=='3'">
			<view class="row">
				<text class="tit">提货人姓名</text>
				<text class="input">{{order.deliveryName}}</text>
			</view>
			<view class="row">
				<text class="tit">手机号码</text>
				<text class="input">{{order.deliveryContactNo}}</text>
			</view>
			<button class="add-btn" @click="save">确认</button>
		</view>
		
	</view>
</template>

<script>
	import wPicker from "@/components/w-picker/w-picker.vue";
	import {
		mapState,
		mapMutations
	} from 'vuex';
	export default {
		data() {
			return {
				current: 0,
				courierName:this.$api.util.courierList[0],	//物流公司
				courierNo:'',	//物流单号
				couriers: this.$api.util.courierList,
				index:0,
				orderNo: '',
				order: '',
				shoppers:[],
				shopper:{}
			};
		},
		components: {
			wPicker
		},
		onLoad(options) {
			this.orderNo = options.orderNo;
			this.inquiryOrder(this.orderNo);
		},
		computed: {
			...mapState(['hasLogin', 'merchantInfo'])
		},
		methods: {
			bindPickerChange(e){
				this.courierName = this.couriers[e.detail.value];
			},
			shopperChange(e){
				this.shopper = this.shoppers[e.detail.value];
				this.index = e.detail.value;
			},
			//查询订单
			inquiryOrder(orderNo) {
				this.$api.request.orderDetail({
					orderNo: orderNo
				}, res => {
					if (res.body.status.statusCode === '0') {
						this.order = res.body.data;
					} else {
						console.log(res.body.status.errorDesc);
					}
				});
			},
			//返回
			back(){
				uni.navigateBack({
					delta:1
				})
			},
			//发货
			save() {
				if(this.order.deliveryType=='1'){
					if (!this.courierName) {
						this.$api.msg('请选择物流公司');
						return;
					} else if (!this.courierNo) {
						this.$api.msg('请输入物流单号');
						return;
					}
				}else if(this.order.deliveryType=='2'){
					this.shopper = this.shoppers[this.index]
					if (!this.shopper.name) {
						this.$api.msg('请选择配送员');
						return;
					}
				}
				
				this.$api.request.deliverOrder({
					orders: [{
						orderNo: this.order.orderNo,
						courierName: this.courierName,
						courierNo: this.courierNo,
						shopperName: this.shopper.name,
						shopperMobileNo: this.shopper.mobileNo,
						shopperSex: this.shopper.sex,
						shopperPhoto: this.shopper.photo
					}]
				}, res => {
					if (res.body.status.statusCode === '0') {
						this.$api.msg('订单已发货');
						setTimeout(function() {
							uni.navigateBack({
								delta:1
							})
						}, 1000);
					} else {
						this.$api.msg(res.body.status.errorDesc);
					}
				});
			},
			//查询配送员
			inquiryShopper() {
				this.$api.request.inquiryMerchantShopper({
					merchantDTO: {
						merchantUuid: this.merchantInfo.merchantUuid
					}
				}, res => {
					if (res.body.status.statusCode === '0') {
						this.shoppers = res.body.data.shoppers;
					} else {
						this.$api.msg(res.body.status.errorDesc);
					}
				});
			}

		},
		mounted() {
			this.inquiryShopper();
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
			width: 180upx;
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
	
	.empty{
		text-align: center;
		font-size: $font-sm;
		color: $font-color-light;
		margin: 40px 0;
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
