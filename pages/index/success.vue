<template>
	<view class="container">
		<view class="card-success">
			<view class="card-success-card">
				<image src="../../static/success.png" />
				<view class="card-success-card-text">提交成功</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			};
		}
	}
</script>

<style lang="scss" scoped>
	.container {
		height: 100vh;
		width: 100vw;
		background-color: rgb(0, 133, 200);
		display: flex;
		flex-direction: row;
		justify-content: center;
	}
	
	.card-success {
		margin: 0 20px;
		margin-top: 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 100vw;
		height: 400rpx;
		background-color: #fff;
		border-radius: 4px;
		box-shadow:
		  2.8px 2.8px 2.2px rgba(0, 0, 0, 0.02),
		  6.7px 6.7px 5.3px rgba(0, 0, 0, 0.028),
		;
	}
	
	.card-success-card {
		height: 200rpx;
		width: 200rpx;
		
		& > image {
			height: 100%;
			width: 100%;
		}
		
		&-text {
			text-align: center;
			line-height: 36rpx;
		}
	}
</style>
