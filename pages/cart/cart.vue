<template>
	<view class="cart-container" v-if="cart.length !== 0">
		<!-- 收收获地址组件 -->
		<my-address></my-address>
		<!-- 商品列表的标题区域 -->
		<view class="cart-title">
			<!-- 左侧图标 -->
			<uni-icons type="shop" size="18"></uni-icons>
			<!-- 右侧文本 -->
			<text class="cart-title-text">购物车</text>
		</view>
		<!-- uni-swipe-action 是最外层包裹性质的容器 -->
		<uni-swipe-action>
		  <block v-for="(goods, i) in cart" :key="i">
		    <!-- uni-swipe-action-item 可以为其子节点提供滑动操作的效果。需要通过 options 属性来指定操作按钮的配置信息 -->
		    <uni-swipe-action-item :right-options="options" @click="swipeActionClickHandler(goods)">
		      <my-goods :goods="goods" :show-radio="true" :show-num="true" @radio-change="radioChangeHandler" @num-change="numberChangeHandler"></my-goods>
			  <!-- <text>測試</text> -->
		    </uni-swipe-action-item>
		  </block>
		</uni-swipe-action>
		
		<!-- 自定义结算组件 -->
		<my-settle></my-settle>
		
		
	</view>
	<!-- 空白购物车的区域 -->
	<view class="empty-cart" v-else>
		<image src="/static/cart_empty@2x.png" class="empty-img"/>
		<text class="tip-text">空空如也</text>
	</view>
</template>

<script>
	import badgeMix from '@/mixins/tabbar-badge.js'
	import {mapState, mapMutations} from 'vuex'
	export default {
		mixins: [badgeMix],
		computed: {
			...mapState('m_cart', ['cart'])
		},
		data() {
			return {
				options: [{
					text: '删除',
					style: {
						backgroundColor: '#C00000'
					}
				}]
			}
		},
		methods: {
			...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount', 'removeGoodsById']),
			radioChangeHandler(e) {
				
				this.updateGoodsState(e)
			},
			numberChangeHandler(e) {
				this.updateGoodsCount(e)
			},
			swipeActionClickHandler(goods) {
				console.log(goods.goods_id)
				console.log(goods)
				this.removeGoodsById(goods.goods_id)
				this.setBadge()
			
			}
		}
	}
</script>

<style lang="scss">
.cart-title {
	height: 40px;
	display: flex;
	align-items: center;
	padding-left: 5px;
	border: 2px solid #efefef;
	border-radius: 24px;
	
	.cart-title-text {
		font-size: 14px;
		margin-left: 10px;
	}
}
.cart-container {
	padding-bottom: 50px;
}
.empty-cart {
	display: flex;
	align-items: center;
	flex-direction: column;
	padding-top: 300rpx;
}
.empty-img {
	width: 180rpx;
	height: 180rpx;
}

.tip-text {
	font-size: 12px;
	color: gray;
	margin-top: 30rpx;
}
</style>
