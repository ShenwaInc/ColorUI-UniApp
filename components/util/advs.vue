<template>
	<view class="wadvs" :class="[extraclass,isPopup?'cu-modal show':'']" v-if="advs.length>0 && showadv">
		<view class="cu-modal-bg" @tap="showadv=false" v-if="isPopup"></view>
		<view :class="isPopup?'cu-dialog':''">
			<view class="wadvs-single" v-if="advs.length==1">
				<block v-if="advs[0].type=='adview'">
					<!--  #ifdef MP-WEIXIN || MP-TOUTIAO || MP-QQ -->
					<ad :unit-id="advs[0].content" ad-intervals="60"></ad>
					<!--  #endif -->
					<!--  #ifdef APP-PLUS -->
					<ad :adpid="advs[0].content" ad-intervals="60"></ad>
					<!--  #endif -->
					<!--  #ifdef MP-BAIDU -->
					<ad :appid="appid" :apid="advs[0].content" updatetime="60"></ad>
					<!--  #endif -->
				</block>
				<view class="grid col-1" @click="naviTo(advs[0].url)" v-else-if="advs[0].type=='pic'">
					<view class="only-img">
						<image mode="widthFix" :src="advs[0].pic"></image>
					</view>
				</view>
				<view class="richtext" v-else>
					<rich-text :nodes="advs[0].content"></rich-text>
				</view>
			</view>
			<view v-else>
				<swiper class="screen-swiper square-dot" :indicator-dots="true" :circular="true" :autoplay="true" interval="3600" duration="500">
					<swiper-item v-for="(item,index) in advs" :key="index" @click="naviTo(item.url)">
						<image :src="item.pic" mode="aspectFill"></image>
					</swiper-item>
				</swiper>
			</view>
			<view class="justify-end" v-if="isPopup">
				<view class="cu-capsule round margin-right-sm">
					<view class="cu-tag" :class="'bg-'+theme.actcolor" @tap="showadv=false">
						<text class="cuIcon-close"></text>
					</view>
					<view class="cu-tag" :class="'line-'+theme.actcolor">
						{{autoclose}}秒后自动关闭
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	var interstitialAd = null;
	export default {
		name: 'advs',
		data(){
			return {
				theme:core.style,
				appid:core.appid,
				showadv:true,
				autoclose:11
			}
		},
		mounted(){
			if(this.isPopup){
				this.autoclose = this.advs.length>1 ? 15 : 11;
				this.autoClose();
			}
		},
		props: {
			advs : {
				type:Array,
				default:function(){
					return [];
				}
			},isPopup:{
				type:Boolean,
				default:false
			},extraclass:{
				type:String,
				default:''
			}
		},methods:{
			naviTo(page,data){
				return core.navito(page,data);
			},
			autoClose(){
				if(this.autoclose==0){
					return this.showadv = false;
				}
				let self = this;
				setTimeout(function(){
					self.autoclose -= 1;
					self.autoClose();
				},1000);
			}
		}
	}
</script>

<style>
	.wadvs-single .only-img image{width: 100%; display: block;}
	.cu-dialog .screen-swiper{min-height: 680upx;}
	.cu-dialog .justify-end{position: absolute; top: 20upx; right: 0;}
</style>
