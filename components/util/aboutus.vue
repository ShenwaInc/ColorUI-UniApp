<template>
	<view class="fullscreen" :class="tabbar||hastabbar?'isfooter':''">
		<swiper class="screen-swiper square-dot" :indicator-dots="true" :circular="true" :autoplay="true" :interval="interval" duration="500" v-if="navkey=='splash'">
			<swiper-item v-for="(item,index) in runshow.data" :key="index">
				<image :src="item" mode="aspectFill"></image>
			</swiper-item>
		</swiper>
		<view class="full-video" v-else-if="navkey=='video'">
			<video :http-cache="true" :src="runshow.data[0]" :autoplay="true" :loop="true" :controls="false" :show-progress="false" object-fit="cover" mobilenet-hint-type="0" :muted="videoMuted">
				<!-- #ifdef APP-PLUS -->
				<cover-view @click="naviTo(runshow.button.url)" :class="runshow.button.style" class="cu-btn lg login-btn">{{runshow.button.text}}</cover-view>
				<!-- #endif -->
			</video>
			<view class="muted-btn text-white shadow-blur" @click="videoMuted=!videoMuted"><text :class="videoMuted?'cuIcon-notificationforbidfill':'cuIcon-notificationfill'"></text></view>
		</view>
		<dynamic @onDynamic="onDynamic" :refresh="refresh" v-else-if="navkey=='moment'"></dynamic>
		<view class="near-page" v-else-if="navkey=='nearby'">
			<nears :autoInit="refresh" navstyle="solid-bottom" @editinfo="doLogin()" @showshare="doLogin()"></nears>
		</view>
		<view class="padding-lr-xl about-bottom" v-if="showBtn">
			<view class="flex flex-direction padding-lr-xl">
				<button @click="naviTo(runshow.button.url)" :class="runshow.button.style" class="cu-btn lg padding-sm margin-tb margin-lr-xl">{{runshow.button.text}}</button>
				<view class="text-center margin-top-sm" v-if="auditmode.status && !haslogin && auditmode.login!=''" @click="naviTo('auth/index')">
					<text class="text-lg" :class="'text-'+btnstyle">{{auditmode.login}}</text>
				</view>
			</view>
		</view>
		<view class="cu-bar tabbar foot" :class="footbg" v-else-if="hastabbar">
			<view class="action" :class="[(navkey==item.key?'cur text-'+theme.actcolor:''),item.extra]" @click="checkoutbar(index)" v-for="(item, index) in runshow.tabbar" :key="index">
				<view :class="item.icon+(navkey==item.key?'fill':'')"></view>
				{{item.name}}
			</view>
		</view>
		<view class="cu-modal login-modal" :class="showLogin?'show':''">
			<view class="cu-dialog bg-white">
				<view class="cu-dialog-login" :style="'background-image:url('+loginBg+');'"></view>
				<view class="cu-bar justify-end">
					<view class="content">请先登录</view>
				</view>
				<view class="padding-bottom-xl text-center">
					<text>这么精彩，快点登录一起玩啊~</text>
				</view>
				<view class="cu-bar">
					<view class="action margin-0 flex-sub text-gray" @tap="showLogin=false">稍后登录</view>
					<view class="action margin-0 flex-sub solid-left text-red" @tap="naviTo('auth/index')">去登录</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	import dynamic from "@/components/util/dynamic.vue"
	import nears from "@/components/util/nears.vue"
	
	export default {
		name: 'aboutus',
		components: {dynamic, nears},
		data(){
			return {
				footbg:core.style.navbg,
				loginBg:core.static('images/bg_modal_login.png'),
				theme:core.style,
				navkey:"splash",
				hastabbar:false,
				showBtn:false,
				showLogin:false,
				videoMuted:true,
				videoMuted:true,
				runShabox:false,
				runTemp:{}
			}
		},
		mounted(){
			this.footbg = core.style.navbg.replace("gradual-", "");
			this.navkey = this.runshow.type;
			if(this.runshow.tabbar.length>0 && ["moment", "nearby"].indexOf(this.navkey)>=0){
				this.hastabbar = true;
			}
			if(!this.NetworkStatus){
				this.navkey = this.runshow.type = "splash";
				this.runshow.data = ['/static/splash.jpg'];
				this.hastabbar = this.tabbar = false;
				this.runshow.tabbar = [];
			}
			if(core.userinfo.uid==0 && (this.navkey=='splash' || this.navkey=='video')){
				this.showBtn = true;
			}
		},
		watch:{
			runshow(){
				this.navkey = this.runshow.type;
				if(this.navkey=='splash' || this.navkey=='video'){
					this.showBtn = true;
				}else{
					this.showBtn = false;
					if(this.runshow.tabbar.length>0){
						this.hastabbar = true;
					}
				}
				if(core.userinfo.uid>0){
					this.showBtn = false;
				}
				// #ifdef APP-PLUS
				if(this.runshow.type=='video'){
					plus.navigator.setStatusBarStyle("light");
				}
				// #endif
				if(!this.NetworkStatus){
					this.runTemp = this.runshow;
					this.runShabox = true;
					this.navkey = this.runshow.type = "splash";
					this.runshow.data = ['/static/splash.jpg'];
					this.hastabbar = this.tabbar = false;
					this.runshow.tabbar = [];
					let self = this;
					core.cacheread("SystemSplashData", function(res){
						if(res.data.length>0){
							self.runshow.data = res.data;
						}
					});
				}else if(this.navkey=="splash"){
					this.doStorage(this.runshow.data);
				}
			},
			NetworkStatus(){
				if(this.runShabox){
					this.runshow = this.runTemp;
					this.navkey = this.runshow.type;
					if(this.runshow.tabbar.length>0){
						this.hastabbar = this.tabbar = true;
					}
				}
			}
		},
		props: {
			haslogin : {
				type:Boolean,
				default:false
			},
			auditmode:{
				type:Object,
				default:function(){
					return {
						status:false,
						login:''
					};
				}
			},
			runshow:{
				type:Object,
				default:function(){
					return {
						type:"splash",
						data:[core.system.splash],
						tabbar:[],
						button:{
							text:'立即体验',
							url:'auth/index',
							style:'round bg-' + core.style.actcolor
						}
					};
				}
			},
			interval:{
				type:Number,
				default:3000
			},
			refresh:{
				type:Number,
				default:0
			},
			NetworkStatus:{
				type:Boolean,
				default:false
			},
			btnstyle:{
				type:String,
				default:"bg-gradual-pink"
			},
			tabbar:{
				type:Boolean,
				default:false
			}
		},
		methods:{
			naviTo(page,data){
				return core.navito(page,data);
			},
			onDynamic(e){
				return this.doLogin();
			},
			checkoutbar(index){
				let navkey = this.runshow.tabbar[index].key;
				if(navkey!='moment' && navkey!='nearby'){
					return this.doLogin();
				}
				this.navkey = navkey;
			},
			doLogin(){
				return this.showLogin = true,1;
			},
			doStorage(data){
				let splashs = [];
				let totals = this.runshow.data.length;
				if(totals==0){
					return core.cacheremove("SystemSplashData");
				}
				for (let i in this.runshow.data) {
					let src = this.runshow.data[i];
					if(src.indexOf('http')!==0){
						totals -= 1;
						continue;
					}
					uni.downloadFile({
						url:src,
						success(res) {
							splashs.push(res.tempFilePath);
							if(splashs.length==totals){
								core.cacheset("SystemSplashData", splashs);
							}
						},
						fail() {
							totals -= 1;
							if(splashs.length==totals){
								core.cacheset("SystemSplashData", splashs);
							}
						}
					});
				}
			}
		}
	}
</script>

<style>
	.about-bottom{position: fixed; bottom: 12vh; width: 100%; left: 0; z-index: 10000;}
	.fullscreen>.screen-swiper{height: 100%;}
	.fullscreen{width: 100%; top: 0px; left: 0; bottom: 0; position: absolute;}
	.fullscreen.isfooter{bottom: 50px;}
	.login-modal .cu-dialog{width: 16rem; padding-top: 3rem; position: relative; overflow: visible;}
	.login-modal .cu-dialog-login{height: 226upx; width: 100%; position: absolute; background: no-repeat center top; background-size: auto 100%; left: 0; top: -96upx;}
	.login-modal .cu-bar .content{font-weight: 600; font-size:1.23rem; color: #333333; font-family:PingFangSC-Semibold,PingFang SC;}
	.full-video{position: absolute; width: 100vw; bottom: 0; z-index: 0; top: 0;}
	.full-video video{width: 100%; height: 100%; position: absolute; z-index: 0; top: 0; bottom: 0;}
	.muted-btn{position: absolute; top: 30upx; right: 30upx; text-align: center; line-height: 50upx; border-radius: 50%; font-size: 36upx; background: rgba(0, 0, 0, 0.2); width: 50upx; height: 50upx;}
	.login-btn{position: absolute; bottom: 12vh; left: 50%; width: 80vw; margin-left: -40vw; line-height: 64upx;}
</style>