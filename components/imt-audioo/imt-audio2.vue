<template>
	<view class="whole">
		<scroll-view  v-show="gecitext" scroll-y="true" class="geciposition" >
			<view>
				<slot name="content"></slot>
			</view>
		</scroll-view>
		<view class="imt-audio">
			<!-- #ifdef H5||MP-WEIXIN -->
			<view class="audio-wrapper">
				<view class="audio-number">{{format(current)}}</view>
				<slider class="audio-slider" :activeColor="color" block-size="16" :value="current" :max="duration" @changing="seek=true,current=$event.detail.value"
				 @change="audio.seek($event.detail.value)"></slider>
				<view class="audio-number">{{format(duration)}}</view>
			</view>
			<!-- #endif -->
			<view class="audio-control-wrapper" :style="{color}">
				<view v-show="dqxh" class="audio-control-switch1" @click="xunhuan">
					<image src="../../../static/xunhuan.png" class="audio-icon"></image>
				</view>
				<view v-show="qxdqxh" class="audio-control-switch1" @click="qxxunhuan">
					<image src="../../../static/qxxunhuan.png" class="audio-icon"></image>
				</view>
				<!-- #ifdef APP-PLUS -->
				<view v-if="control" :style="{borderColor:color}" @click="prev">
					<image src="../../../static/shangyiqu.png" class="audio-icon1"></image>
				</view>
				<view v-show="!paused" :style="{borderColor:color}" @click="pause">
					<image src="../../../static/pause.png" class="audio-control-switch audio-icon1"></image>
				</view>
				<view v-show="paused" :style="{borderColor:color}" @click="play">
					<image src="../../../static/play.png" class="audio-control-switch audio-icon1"></image>
				</view>
				<view v-if="control" :style="{borderColor:color}" @click="next">
					<image src="../../../static/xiayiqu.png" class="audio-icon1"></image>
				</view>
				<!-- #endif -->
				<!-- #ifdef H5||MP-WEIXIN -->
				<view class="audio-control audio-control-prev" v-if="control" :style="{borderColor:color}" @click="prev">&#xe601;</view>
				<view class="audio-control audio-control-switch" :class="{audioLoading:loading}" :style="{borderColor:color}" @click="audio.paused?play():audio.pause()">{{loading?'&#xe600;':(paused?'&#xe865;':'&#xe612;')}}</view>
				<view class="audio-control audio-control-next" v-if="control" :style="{borderColor:color}" @click="next">&#xe601;</view>
				<!-- #endif -->
				<view v-show="showgeci" class="audio-control-switch2" @click="geci">
					<image src="../../../static/geci.png" class="audio-icon"></image>
				</view>
				<view v-show="shutdowngeci" class="audio-control-switch2" @click="qxgeci">
					<image src="../../../static/qxgeci.png" class="audio-icon"></image>
				</view>
			</view>
		</view>
	</view>
	
</template>

<script>
	export default {
		data() {
			return {
				audio: uni.createInnerAudioContext(),
				current: 0, //????????????(s)
				duration: 0, //?????????(s)
				paused: true, //????????????????????????
				loading: false, //????????????????????????
				seek: false, //????????????????????????
				dqxh: true, //?????????????????????
				qxdqxh: false, //??????????????????
				showgeci: true, //???????????????
				shutdowngeci: false ,//????????????
				gecitext:false,//???????????????
			}
		},
		props: {
			src: String, //????????????
			autoplay: Boolean, //??????????????????
			looping: Boolean, //??????????????????
			continue: Boolean, //??????????????????????????????????????????????????????@next??????
			control: {
				type: Boolean,
				default: true
			}, //?????????????????????/???????????????
			color: {
				type: String,
				default: '#169af3'
			} //?????????
		},
		methods: {
			//??????prev??????
			prev() {
				this.$emit('prev')
				uni.showToast({
					title: "?????????",
					icon: "none"
				});
			},
			//??????next??????
			next() {
				this.$emit('next')
				uni.showToast({
					title: "?????????",
					icon: "none"
				});
			},
			//???????????????
			format(num) {
				return '0'.repeat(2 - String(Math.floor(num / 60)).length) + Math.floor(num / 60) + ':' + '0'.repeat(2 - String(
					Math.floor(num % 60)).length) + Math.floor(num % 60)
			},
			//??????????????????
			play() {
				//#ifdef H5||MP-WEIXIN
				this.loading = true
				//#endif
				// #ifdef APP-PLUS
				this.paused = false
				//#endif
				this.audio.play()
			},
			//??????????????????
			// #ifdef APP-PLUS
			pause() {
				this.audio.pause()
				this.paused = true
			},
			//#endif
			//??????????????????
			xunhuan() {
				this.audio.loop = true
				this.dqxh = false
				this.qxdqxh = true
				uni.showToast({
					title: "??????????????????",
					icon: "none"
				});
			},
			//??????????????????
			qxxunhuan() {
				this.audio.loop = false
				this.dqxh = true
				this.qxdqxh = false
				uni.showToast({
					title: "??????????????????",
					icon: "none"
				});
			},
			//????????????
			geci() {
				this.showgeci = false
				this.shutdowngeci = true
				this.gecitext=true
			},
			//????????????
			qxgeci() {
				this.showgeci = true
				this.shutdowngeci = false
				this.gecitext=false
			},
		},
		created() {
			if (this.src) {
				this.audio.src = this.src
				this.autoplay && this.play()
			}
			this.audio.obeyMuteSwitch = false
			//????????????????????????
			this.audio.onTimeUpdate(() => {
				if (!this.seek) {
					this.current = this.audio.currentTime
				}
				if (!this.duration) {
					this.duration = this.audio.duration
				}
			})
			//??????????????????
			this.audio.onPlay(() => {
				this.paused = false
				this.loading = false
			})
			//??????????????????
			this.audio.onPause(() => {
				this.paused = true
			})
			//??????????????????
			this.audio.onEnded(() => {
				//#ifdef H5||MP-WEIXIN
				if (this.continue) {
					this.next()
				} else {
					this.paused = true
					this.current = 0
				}
				//#endif
				//#ifndef APP-PLUS
				this.next()
				//#endif
			})
			//??????????????????????????????
			this.audio.onSeeked(() => {
				this.seek = false
			})
		},
		beforeDestroy() {
			this.audio.destroy()
		},
		watch: {
			src(src, old) {
				this.audio.src = src
				this.current = 0
				this.duration = 0
				if (old || this.autoplay) {
					this.play()
				}
			}
		}
	}
</script>

<style>
	@font-face {
		font-family: 'icon';
		src: url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/1c90d860-2a35-11eb-899d-733ae62bed2f.eot');
		src: url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/1c90d860-2a35-11eb-899d-733ae62bed2f.eot?#iefix') format('embedded-opentype'),
			url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/1bbebb50-2a35-11eb-8a36-ebb87efcf8c0.woff2') format('woff2'),
			url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/892b6d50-2a35-11eb-97b7-0dc4655d6e68.woff') format('woff'),
			url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/ab0559e0-2a35-11eb-b680-7980c8a877b8.ttf') format('truetype'),
			url('https://vkceyugu.cdn.bspapp.com/VKCEYUGU-aliyun-hjinhmzy11qaa4ebe2/e2502ab0-2a35-11eb-b997-9918a5dda011.svg#iconfont') format('svg');
	}

	.imt-audio {
		width: 100%;
		padding: 30upx 0;
		background: #fff;
		border-radius: 20upx;
	}

	.audio-wrapper {
		display: flex;
		align-items: center;
	}

	.audio-number {
		width: 120upx;
		font-size: 24upx;
		line-height: 1;
		color: #333;
		text-align: center;
	}

	.audio-slider {
		flex: 1;
		margin: 0;
	}

	.audio-control-wrapper {
		margin-top: 20upx;
		display: flex;
		justify-content: center;
		align-items: center;
		font-family: "icon" !important;
	}

	.audio-control {
		font-size: 32upx;
		line-height: 1;
		border: 4upx solid;
		border-radius: 50%;
		padding: 16upx;
	}

	.audio-control-next {
		transform: rotate(180deg);
	}

	.audio-control-switch {
		font-size: 40upx;
		margin: 0 80upx;
	}

	.audio-control-switch1 {
		font-size: 40upx;
		margin-right: 80upx;
		/* margin: 0 50upx; */
	}

	.audio-control-switch2 {
		font-size: 40upx;
		margin-left: 80upx;
		/* margin: 0 50upx; */
	}

	.audio-icon {
		width: 50upx;
		height: 50upx;
	}

	.audio-icon1 {
		width: 85upx;
		height: 85upx;
	}

	.audioLoading {
		animation: loading 2s;
		animation-iteration-count: infinite;
		animation-timing-function: linear;
	}

	@keyframes loading {
		to {
			transform: rotate(360deg);
		}
	}
	.geciposition{
		/* #ifdef APP-PLUS */
		margin-top: 78upx;
		/* #endif */
		width: 85%;
		height: 650upx;
		position: fixed;
		color: #fff;
		font-size: 35upx;
		top: 70upx;
		background-color: rgba(0, 0, 0, 0.5);
	}
	.whole{
		display: flex;
		flex-direction: column;
		align-items: center;
	}
</style>
