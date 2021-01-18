<template>
	<form class='loginView'>
		<view class="history-box">
			<view class="history-list" v-if="historyList && historyList.length">
				<view class="item" v-for="(item, index) in historyList" :key="index" @click="deleteItem(item)">
					<view class="time">
						<view class="cell">{{showTime(item.createTime)}}</view>
						<view class="cell">
							<view class="label">油价</view>
							<view class="value">{{item.unitPrice}}</view>
						</view>
						<view  class="cell">
							<view class="label">加油费用</view>
							<view class="value">{{item.totalPrice}}</view>
						</view>
					</view>
					<view class="content">
						<view  class="cell">
							<view class="label">公里数</view>
							<view class="value">{{item.km}}</view>
						</view>
						<view  class="cell">
							<view class="label">百公里油耗</view>
							<view class="value">{{item.oil100}}升</view>
						</view>
						<view  class="cell">
							<view class="label">每公里耗费</view>
							<view class="value">{{item.money1km}}元</view>
						</view>
					</view>
				</view>
			</view>
			<view v-else class="common-no-data-box">
				<NoData :text="loading?'加载中...':'暂无数据'"></NoData>
			</view>
		</view>
		<ad unit-id="adunit-2c56a0998bfafd4e" ad-type="video" ad-theme="black"></ad>
	</form>
</template>

<script>
	import NoData from '../components/no-data'
	import {
		mapState,
		mapMutations
	} from 'vuex'
	import * as util from '../../common/util'
	export default {
		components: {
			NoData
		},
		data() {
			return {
				unitPrice: '',
				totalPrice: '',
				km: '',
				oil100: 0,
				money1km: 0,
				historyList: [],
				loading: false
			};
		},
        computed: mapState(['userInfo']),
		methods: {
			...mapMutations(['getUserInfo','setStateData']),
			showTime (time) {
				return util.dateFormat(new Date(time).getTime(), 'yyyy-MM-dd')
			},
			// 删除
			deleteItem (item) {
				uni.showModal({
					title: '提示',
					content: `确认删除该条记录吗？`,
					showCancel: true,
					success: async (res3) => {
						if (res3.confirm) {
							let res = await wx.cloud.callFunction({
								name: 'delDataFromCould',
								data: {
									dbName: 'userOilCompute',
									primaryKey: '_id',
									list: [item]
								}
							})
							if (res.errMsg === 'cloud.callFunction:ok') {
								uni.showToast({
									title: `删除成功！`,
									icon: 'none',
								});
							}
							this.getOilHistory()
						} else if (res3.cancel) {
						}
					}
				});
			},
			// 获取历史油耗
			async getOilHistory () {
				this.loading = true
				let res = await wx.cloud.callFunction({
					name: 'getDbListData',
					data: {
						dbName: 'userOilCompute',
						pageNo: 1,
						pageSize: 50,
						limitType: 1,
						orderName: 'createTime',
						orderType: 'desc',
						params: {
						},
					}
				})
				this.loading = false
				if (res.errMsg === 'cloud.callFunction:ok') {
					this.historyList = res.result.data
				}
			}
		},
		// 加了这个页面才可以被分享
		onShareAppMessage () {
		},
		onShow () {
			this.getOilHistory()
		},
		async onLoad() {
		}
	}
</script>

<style lang="stylus" scoped>
	@import "../../uni.styl"
	.input-view-item
		border-bottom 1px solid $uni-border-color
	.result-box
		display flex
		padding 20px 10px 10px;
		justify-content space-between
	.card-item
		display flex
		align-items center
		.label
			margin-right 5px
		.value
			font-size 20px
			color $uni-color-success
	.history-box
		display block
		border-radius 5px;
		box-shadow $uni-box-shadow
		margin 0 5px 20px
		.title
			justify-content center
			line-height 50px
			font-size 20px
			border-bottom 1px solid $uni-border-color

		.history-list
			display block
			.item
				display block
				padding 5px 10px
				margin 10px
				border-radius 5px
				background-color #1c1c1d
				.time
					display flex
					justify-content space-between
					border-bottom 1px solid $uni-border-color
				.content
					display flex
					justify-content space-between
				.cell
					flex 1
					font-size 14px
					align-items center
					padding 3px 0
					.label
						font-size 12px
					.value
						font-size 14px
						color $uni-color-success
</style>
