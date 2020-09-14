<template>
	<view style="height: 100%;">
		<image style="width: 100%;height: 35%;" src="../../static/image/lok1.png"></image>
		<view class="soy">
				<view><image src="../../static/image/lok2.png"></image></view>
				<text>恭喜您,入驻申请已提交。</text>
				<view style="margin-top: 20px;font-size: 12px;color: #fff">请耐心等待审核，审核结果将通过手机短信发送给你, 请注意查收。</view>
		</view>
		<button  class="confirm-btn" @click="complete">完成</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
					
				}
		},
		onLoad(res) {
			
		},
		methods: {
			complete(){
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
		}
	}
</script>

<style lang='scss'>
	page {
		height: 100%;
		/* background-color: #4b70c1; */
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
	.soy{
		width: 100%;
		height: 20%;
		position: fixed;
		top: 2%;
		text-align: center;
	}
	.soy image{
		width: 85px;
		height: 68px;
	}
	.soy text{
		font-size: 16px;
		color: #FFFFFF;
	}
</style>
