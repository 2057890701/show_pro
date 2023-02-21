<template>
	<view>
		<view class="search-box">
			<!-- 使用自定义的搜索组件 -->
			<my-search :bgcolor="'#c00000'" :radius="15" @click="gotoSearch"></my-search>
		</view>
		<view class="scroll-view-container ">
			<!-- 左侧的滚动视图区域 -->
			<scroll-view class="left-scroll-view" scroll-y style="height: 100vh;">
				<view :class="['left-scroll-view-item', i === active ? 'active' : '']" v-for="(item, i) in cateList" :key="i" @click="activeChanged(i, item.children)">
					{{ item.cat_name }}
				</view>
			</scroll-view>
			<!-- 右侧的滚动视图区域 -->
			<scroll-view class="right-scroll-view" scroll-y :style="{ height: wh + 'px' }" :scroll-top="scrollTop">
				<view class="cate-lv2" v-for="(item2, i2) in cateLevel2" :key="i2">
					<view class="cate-lv2-title">/ {{ item2.cat_name }} /</view>
					<!-- 动态渲染三级分类的列表数据 -->
					<view class="cate-lv3-list">
						<!-- 三级分类 Item 项 -->
						<view class="cate-lv3-item" v-for="(item3, i3) in item2.children" :key="i3" @click="gotoGoodsList(item3)">
							<!-- 图片 -->
							<image :src="item3.cat_icon"></image>
							<!-- 文本 -->
							<text>{{ item3.cat_name }}</text>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			wh: 0,
			cateList: [],
			active: 0,
			cateLevel2: [],
			// 滚动条距离顶部的距离
			scrollTop: 0
		};
	},
	onLoad() {
		// 获取当前系统的信息
		const sysInfo = uni.getSystemInfoSync();
		// 为 wh 窗口可用高度动态赋值
		this.wh = sysInfo.windowHeight - 50;
		this.getCateList();
	},
	methods: {
		// 跳转到分包中的搜索页面
		gotoSearch() {
			uni.navigateTo({
				url: '/subpkg/search/search'
			});
		},
		async getCateList() {
			// 发起请求
			const { data } = await uni.$http.get('/api/public/v1/categories');
			console.log(data);
			// 判断是否获取失败
			if (data.meta.status !== 200) return uni.$showMsg();
			// 转存数据
			this.cateList = data.message;
			// 为二级分类赋值, 分类页面刚打开第一个默认选中的数据
			this.cateLevel2 = data.message[0].children;
		},
		activeChanged(i, children) {
			this.active = i;
			this.cateLevel2 = children;
			console.log(this.cateLevel2);
			// 让 scrollTop 的值在 0 与 1 之间切换
			this.scrollTop = this.scrollTop ? 0 : 1;
		},
		gotoGoodsList(item) {
			uni.navigateTo({
				url: '/subpkg/goods_list/goods_list?cid=' + item.cat_id
			});
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
.scroll-view-container {
	display: flex;
}
.left-scroll-view {
	width: 120px;
}
.left-scroll-view-item {
	text-align: center;
	line-height: 60px;
	font-size: 12px;
	&.active {
		position: relative;
		background-color: #fff;
		&::before {
			content: '';
			width: 3px;
			height: 50px;
			background-color: red;
			position: absolute;
			left: 0;
			top: 50%;
			transform: translateY(-50%);
		}
	}
}
.cate-lv2-title {
	font-size: 12px;
	font-weight: bold;
	text-align: center;
	padding: 15px 0;
}
.cate-lv3-list {
	display: flex;
	flex-wrap: wrap;

	.cate-lv3-item {
		width: 33.33%;
		margin-bottom: 10px;
		display: flex;
		flex-direction: column;
		align-items: center;

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
