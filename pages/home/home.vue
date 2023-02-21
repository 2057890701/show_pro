<template>
	<view>
		<!-- 使用自定义的搜索组件 -->
		<view class="search-box"><my-search @click="gotoSearch"></my-search></view>
		<!-- 轮播图区域 -->
		<swiper indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
			<!-- 循环渲染轮播图的 item 项 -->
			<swiper-item v-for="(item, i) in swiperList" :key="i">
				<navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id">
					<!-- 动态绑定图片的 src 属性 -->
					<image :src="item.image_src"></image>
				</navigator>
			</swiper-item>
		</swiper>
		<!-- // 导航 -->
		<view class="nav-list">
			<view @click="navClickHandler(item)" class="nav-item" v-for="(item, i) in navList" :key="i"><image :src="item.image_src" alt="" class="nav-img" /></view>
		</view>
		<!-- 楼层 -->
		<view class="floor_list">
			<view class="floor_item" v-for="(item, i) in floorList" :key="i">
				<image class="floor-title" :src="item.floor_title.image_src" mode=""></image>
				<view class="floor-img-box">
					<navigator class="left_img-box" :url="item.product_list[0].url">
						<image :src="item.product_list[0].image_src" :style="{ width: item.product_list[0].image_width + 'rpx' }" mode="widthFix"></image>
					</navigator>
					<view class="right_img_box">
						<navigator class="right-img-item" v-for="(item2, i2) in item.product_list" :key="i2" v-if="i2 !== 0" :url="item2.url">
							<image :src="item2.image_src" mode="widthFix" :style="{ width: item2.image_width + 'rpx' }"></image>
						</navigator>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			swiperList: [],
			//分类导航的数据列表
			navList: [],
			floorList: []
		};
	},
	onLoad() {
		//  在小程序页面刚加载的时候，调用获取轮播图数据的方法
		this.getSwiperList();
		this.getNavList();
		this.getFloorList();
	},
	methods: {
		// 跳转到分包中的搜索页面
		gotoSearch() {
			uni.navigateTo({
				url: '/subpkg/search/search'
			});
		},
		async getFloorList() {
			const { data } = await uni.$http.get('/api/public/v1/home/floordata');
			console.log(data);
			if (data.meta.status !== 200) {
				return uni.$showMsg();
			}
			// 通过双层 forEach 循环，处理 URL 地址
			data.message.forEach(floor => {
				floor.product_list.forEach(prod => {
					prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1];
				});
			});
			this.floorList = data.message;
		},
		async getSwiperList() {
			// 发起请求
			const { data } = await uni.$http.get('/api/public/v1/home/swiperdata');
			// console.log(data);
			// 请求失败
			if (data.meta.status !== 200) {
				return uni.$showMsg();
			}
			// 请求成功，为 data 中的数据赋值
			this.swiperList = data.message;
		},
		async getNavList() {
			const { data } = await uni.$http.get('/api/public/v1/home/catitems');
			// console.log(data);
			if (data.meta.status !== 200) return uni.$showMsg();
			this.navList = data.message;
		},
		// nav-item 项被点击时候的事件处理函数
		navClickHandler(item) {
			if (item.name === '分类') {
				uni.switchTab({
					url: '/pages/cate/cate'
				});
			}
		}
	}
};
</script>

<style lang="scss">
.search-box {
	// 设置定位效果为“吸顶”
	position: sticky;
	// 吸顶的“位置”
	top: 0;
	// 提高层级，防止被轮播图覆盖
	z-index: 999;
}
swiper {
	height: 330rpx;

	.swiper-item,
	image {
		width: 100%;
		height: 100%;
	}
}
.nav-list {
	display: flex;
	justify-content: space-around;
	margin: 15px 0;
}
.nav-img {
	width: 128rpx;
	height: 140rpx;
}
.floor-img-box {
	display: flex;
	padding-left: 10rpx;
}
.floor-title {
	height: 60rpx;
	width: 100%;
	// display: flex;
}
.right_img_box {
	display: flex;
	justify-content: space-around;
	flex-wrap: wrap;
}
</style>
