<template>
	<view class="my-settle-container">
		<!-- 全选 -->
		<label class="radio" @click="changeAllState">
			<radio color="#C00000" :checked="isFullCheck"/><text>全选</text>
		</label>
		<!-- 合计 -->
		<view class="amount-box">
			合计:<text class="amount">￥{{checkedGoodsAmount}}</text>
		</view>
		<!-- 结算按钮 -->
		<view class="btn-settle" @click="settlement">结算({{checkedCount}})</view>
	</view>
</template>

<script>
	import {mapGetters, mapMutations, mapState} from 'vuex'
	export default {
		name:"my-settle",
		data() {
			return {
				//倒计时的秒数
				seconds: 3,
				//定时器的id
				timer: null
			};
		},
		computed: {
			...mapGetters('m_cart', ['checkedCount', 'total', 'checkedGoodsAmount']),
			...mapGetters('m_user', ['addstr']),
			...mapState('m_user', ['token']),
			...mapState('m_cart', ['cart']),
			isFullCheck() {
				return this.total === this.checkedCount
			}
		},
		methods: {
			...mapMutations('m_cart', ['updateAllGoodsState']),
			...mapMutations('m_user', ['updateRedirectInfo']),
			changeAllState() {
				this.updateAllGoodsState(!this.isFullCheck)
			},
			//用户点击结算按钮
			settlement() {
				
				if (!this.checkedCount) return uni.$showMsg('请选择要结算的商品')
				
				if (!this.addstr)  return uni.$showMsg('请选择收货地址')
				
				// if (!this.token) return uni.$showMsg('请先登录')
				if (!this.token) return this.delayNavigate()
				
				this.payOrder()
			},
			async payOrder() {
				//1. 创建订单
				
				//1.1 组织订单的信息对象
				const orderInfo = {
					order_price: 0.01,
					consignee_addr: this.addstr,
					goods: this.cart.filter(x => x.goods_state).map(x => ({
						goods_id: x.goods_id,
						goods_number: x.goods_count,
						goods_price: x.goods_price
					})),
				}
				//1.2 发起请求创建订单
				const {data : res} = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
				if (res.meta.status !== 200) return uni.$showMsg('创建订单失败')
				//1.3 得到服务器响应的“订单编号”
				const orderNumber = res.message.order_number
				console.log(orderNumber)
				
				
				//2. 订单预支付
				//2.1 发起请求获取订单的支付信息
				const {data: res2} = await uni.$http.post('/api/public/v1/my/orders/req_unifieldorder', {
					order_number: orderNumber
				})
				if (res2.meta.status !== 200) return uni.$showMsg('预付订单失败')
				const payInfo = res2.message.pay
				
				console.log(payInfo)
				
				//3. 发去微信支付
				const [err, succ] = await uni.requestPayment(payInfo)
				const {data: res3}= uni.$http.post('/api/public/v1/my/orders/chkOrder', {order_number : orderNumber})
				if (res3.meta.status !== 200) return uni.$showMsg('订单未支付')
				uni.showToast({
					title: '订单支付完成',
					icon: 'success'
				})
			},
			//延时导航到my页面
			delayNavigate () {
				this.showTips(this.seconds)
				this.timer = setInterval(() => {
					this.seconds--
					
					if (this.seconds <= 0) {
						clearInterval(this.timer)
						uni.switchTab({
							url: '/pages/my/my',
							success: () => {
								console.log("成功了")
								this.updateRedirectInfo({
									opentype: 'switchTab',
									from: '/pages/cart/cart'
								})
							}
						})
						this.seconds = 3
						return
					}
					this.showTips(this.seconds)
				}, 1000)
			},
			showTips(n) {
				uni.showToast({
					icon: 'none',
					title: '请登录后再结算！' + n + '秒之后自动跳转到到登录页',
					mask: true,
					duration: 1500
				})
			}
		}
	}
</script>

<style lang="scss">
.my-settle-container {
	position: fixed;
	bottom: 0;
	left: 0;
	width: 100%;
	height: 50px;
	background-color: white;
	display: flex;
	justify-content: space-between;
	align-items: center;
	font-size: 14px;
	padding-left: 5px;
	.radio {
		display: flex;
		align-items: center;
	}
	
	.amount-box {
		.amount {
			font-weight: bold;
			color: #C00000;
		}
	}
	
	.btn-settle {
		background-color: #C00000;
		height: 50px;
		color: white;
		line-height: 50px;
		padding: 0 10px;
		min-width: 100px;
		text-align: center;
	}
}
</style>