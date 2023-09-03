<template>
	<view class="swaloading flex justify-center align-center" :style="'top:'+pagetop+'px;height:calc(100% - '+heightless+'px)'" :class="'bg-'+bgColor">
		<view class="text-empty">
			<view class="cu-load loading load-cuIcon">加载中</view>
		</view>
	</view>
</template>

<script>
	
	export default {
		name: 'swaload',
		data() {
			return {
				heightless: 0,
				pagetop:0
			};
		},
		mounted() {
			if(this.hasTopbar){
				this.pagetop = this.StatusBar + this.CustomBar;
				this.heightless += this.pagetop;
			}
			if(this.hasToolbar){
				this.heightless += 75;
			}
			if(this.extratop>0){
				this.pagetop += this.extratop;
				this.heightless += this.extratop;
			}
		},
		props: {
			bgColor: {
				type: String,
				default: 'white'
			},
			hasTopbar:{
				type: [Boolean, String],
				default: false
			},
			hasToolbar:{
				type: [Boolean, String],
				default: false
			},
			extratop:{
				type: Number,
				default: 0
			}
		}
	}
</script>

<style>
	.swaloading{position: absolute;z-index: 998; width: 100%; height: 100%; top: 0; left: 0;}
</style>