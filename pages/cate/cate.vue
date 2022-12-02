<template>
	<view>
		<!-- 使用自定义的组件 -->
		<my-search :bgcolor="'#000000'" :radius="17" @click="gotoSearch"></my-search>
		<view class="scroll-view-container">
			<!-- 左侧滑动区 -->
			<scroll-view class="left-scroll-view" scroll-y="true" :style="{height: wh + 'px'}">
				<block v-for="(item, i) in cateList" :key="i">
					<view :class="['left-scroll-view-item', i === active ? 'active' : '']" @click="activeChanged(i)">{{item.cat_name}}</view>
				</block>
				
				
			</scroll-view>

			<!-- 右侧滑动区 -->
			<scroll-view class="right-scroll-view" scroll-y="true" :style="{height: wh + 'px'}" :scroll-top="scrollTop">
				<view class="cate-lv2" v-for="(item2, i2) in cateLevel2" :key="i2">
					<!-- 二级分类的标题 -->
					<view class="cate-lv2-title">/ {{item2.cat_name}} /</view>
					<!-- 二级分类下的三级分类列表 -->
					<view class="cate-lv3-list">
						<view class="cate-lv3-item" v-for="(item3, i3) in item2.children" :key="i3" @click="gotoGoodsList(item3)">
							<image :src="item3.cat_icon.replace('dev', 'web')"></image>
							<text>{{item3.cat_name}}</text>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import badgeMix from '@/mixins/tabbar-badge.js'
	export default {
		mixins: [badgeMix],
		data() {
			return {
				sysInfo: {},
				//当前设备可用高度
				wh: 0,
				cateList: [],
				active: 0,
				//二级分类的列表
				cateLevel2: [],
				scrollTop: 0
			};
		},
		onLoad() {
			const sysInfo = uni.getSystemInfoSync();
			console.log(sysInfo)
			this.sysInfo = sysInfo;
			this.wh = sysInfo.windowHeight - 50 
			
			this.getCateList()
		},
		methods: {
			//获取分类列表的数据
			async getCateList() {
				const {data: res} = await uni.$http.get('/api/public/v1/categories')
				if (res.meta.status !== 200) return uni.$showMsg()
				
				this.cateList = res.message
				this.cateLevel2  = res.message[0].children
			},
			activeChanged(i) {
				this.active = i
				this.cateLevel2 = this.cateList[i].children
				this.scrollTop = this.scrollTop == 0 ?1 : 0
			},
			gotoGoodsList(item) {
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?cid=' + item.cat_id
				})
			},
			gotoSearch() {
				uni.navigateTo({
					url: '/subpkg/search/search'
				})
			}
			
		}
	}
</script>

<style lang="scss">
	.scroll-view-container {
		display: flex;


		.left-scroll-view {
			width: 120px;

			.left-scroll-view-item {
				background-color: #F7F7F7;
				line-height: 60px;
				text-align: center;
				font-size: 12px;


				&.active {
					background-color: #ffffff;
					position: relative;

					&::before {
						content: ' ';
						display: block;
						width: 3px;
						height: 30px;
						background-color: #c00000;
						position: absolute;
						top: 50%;
						left: 0;
					}
				}
			}
		}
	}
	.cate-lv2-title {
		font-size: 12px;
		text-align: center;
		font-weight: bold;
		padding: 15px 0;
		
		
		
	}
	.right-scroll-view {
		background-color: #ffffff;
	}
	.cate-lv3-list {
		display: flex;
		flex-wrap: wrap;
		
		
		.cate-lv3-item {
			width: 33.33%;
			flex-direction: column;
			display: flex;
			justify-content: center;
			align-items: center;
			margin-bottom: 10px;
			image {
				width: 60px;
				height: 60px;
			}
			
			text {
				font-size: 12px;
			}
		}
		
	}
</style>
