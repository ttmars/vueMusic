<template>
	<view>
		<!-- 搜索条 -->
		<view class="searchTopBox">
			<view class="searchBoxRadius">
				<view class="grace-search-icon searchBoxIcon"></view>
				<input class="searchBoxIpt" type="search" v-model="ipt" @confirm="searchNow($event)" placeholder="歌名/歌手"></input>
			</view>
		</view>
		<!-- 搜索历史 -->
		<view v-if=false class="searchBotBox">
			<view class="ov">
				<view class="fl">搜索历史</view>
				<view @tap="clearKey" class="fr grace-more-r grace-search-remove">删除</view>
			</view>
			<view class="searchHistoryBox">
				<view class="searchHistoryBoxItem" v-for="(item,index) in searchKey" :key='index'>
					{{item}}
				</view>
			</view>
		</view>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				searchKey: [],
				ipt: '',
				searchClose: true,
			}
		},
		onShow() {
			var vv = uni.getStorageSync('searchLocal');
			var arr = vv.split("-");
			this.searchKey = arr;
		},
		methods: {

			clearKey: function() {
				var that = this;
				uni.showModal({
					title: '提示',
					content: '点击确定将删除所有历史信息，确定删除吗？',
					success: function(res) {
						if (res.confirm) {
							that.searchKey = [];
							uni.setStorage({
								key: 'searchLocal',
								data: that.searchKey
							});
							
						} else if (res.cancel) {

						}
					}
				});

			},
			searchNow: function(e) {
				if (this.ipt == '') {
					uni.showToast({
						title: '未输入搜索关键字',
						icon: 'none',
						duration: 1000
					});
					return false;
				}
				var that = this;
				var newArr = that.searchKey;
				newArr.push(this.ipt);
				if (newArr.length == 6){
					newArr.shift()
				}
				this.searchKey = newArr;
				var newStr = newArr.join('-');
				uni.setStorage({
					key: 'searchLocal',
					data: newStr
				});
				this.$emit('flush_data', this.ipt)
				// uni.request({
				// 	url:'http://39.101.203.25/' + this.search_type + '/api?keyword=' + this.ipt,
				// 	method:'GET',
				// 	data:{},
				// 	success: (res) => {
				// 		this.$emit('flush_data', res)
				// 	}
				// });
			}
		}
	}
</script>
<style>
	page {
		background: #FFF;
	}

	.ov {
		overflow: hidden;
	}

	.fl {
		float: left;
	}

	.fr {
		float: right;
	}

	.searchTopBox {
		width: 100%;
		background-color: #877383;
		height: 100upx;
		box-sizing: border-box;
		padding-top: 15upx;
	}

	.searchBoxRadius {
		width: 90%;
		height: 70upx;
		background-color: #ffffff;
		margin-left: 5%;
		overflow: hidden;
		border-radius: 35upx;
	}

	.searchBoxIcon {
		font-size: 40upx;
		margin-top: 20upx;
		margin-left: 20upx;
		float: left;
	}

	.searchBoxIpt {
		height: 70upx;
		line-height: 70upx;
		margin-left: 20upx;
		float: left;
	}

	.searchBotBox {
		width: 100%;
		margin-top: 30upx;
		padding: 15upx 3%;
		box-sizing: border-box;
	}

	.searchHistoryBox {
		width: 100%;
		box-sizing: border-box;
		overflow: hidden;
		margin-top: 40upx;
	}

	.searchHistoryBoxItem {
		float: left;
		font-size: 26upx;
		color: #666;
		line-height: 46upx;
		height: 46upx;
		padding: 0 20upx;
		border-radius: 23upx;
		margin-left: 15upx;
		margin-bottom: 20upx;
		border: 1px solid #ccc;
	}
</style>
