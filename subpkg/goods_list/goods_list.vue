<template>
	<view>
		<view class="goods-list">
			<view v-for="(item, i) in goodsList" :key="i" @click="gotoDetail(item)">
				<!-- 为 my-goods 组件动态绑定 goods 属性的值 -->
				<my-goods :goods="item"></my-goods>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			// 请求参数对象
			queryObj: {
				// 查询关键词
				query: '',
				// 商品分类Id
				cid: '',
				// 页码值
				pagenum: 1,
				// 每页显示多少条数据
				pagesize: 10
			},
			// 商品列表的数据
			goodsList: [],
			// 总数量，用来实现分页
			total: 0,
			isLoading: false
		};
	},
	onLoad(options) {
		console.log(options);
		// 将页面参数转存到 this.queryObj 对象中
		this.queryObj.query = options.query || '';
		this.queryObj.cid = options.cid || '';
		// 调用获取商品列表数据的方法
		this.getGoodsList();
	},
	methods: {
		gotoDetail(item) {
			uni.navigateTo({
				url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
			});
		},
		// 获取商品列表数据的方法
		async getGoodsList(cb) {
			// 发起请求
			console.log(this.queryObj, 111);
			this.isLoading = true;
			const { data } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj);
			this.isLoading = false;
			// 只要数据请求完毕，就立即按需调用 cb 回调函数
			cb && cb();
			if (data.meta.status !== 200) return uni.$showMsg();
			// 为数据赋值：通过展开运算符的形式，进行新旧数据的拼接
			this.goodsList = [...this.goodsList, ...data.message.goods];
			this.total = data.message.total;
		}
	},
	// 触底的事件
	onReachBottom() {
		if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！');
		if (this.isloading) return;
		// 让页码值自增 +1
		this.queryObj.pagenum += 1;
		// 重新获取列表数据
		this.getGoodsList();
	},
	// 下拉刷新的事件
	onPullDownRefresh() {
		// 1. 重置关键数据
		this.queryObj.pagenum = 1;
		this.total = 0;
		this.isloading = false;
		this.goodsList = [];
		// 2. 重新发起请求
		this.getGoodsList(() => uni.stopPullDownRefresh());
	}
};
</script>

<style lang="scss"></style>
