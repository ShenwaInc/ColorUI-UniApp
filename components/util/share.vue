<template>
	<view class="sharepannel" :class="'share-'+platform">
		<view class="cu-modal bottom-modal sharemodal" :class="showshare?'show':''">
			<view class="cu-modal-bg" @click="doclose()"></view>
			<!-- #ifdef H5 -->
			<view class="share-wechat padding-lr" v-if="inwechat">
				<view class="bg-arrow"></view>
				<view class="text-white padding-xl center">
					<text class="text-xl">1.点击右上角的</text><button class="cu-btn line-white sm margin-left-sm margin-right-sm"><text class="cuIcon-more text-xl"></text></button>
					<text class="text-xl">按钮</text>
					<view class="text-xl">
						2.点击【发送给朋友】或【分享到朋友圈】
					</view>
					<view class="text-xl">将精彩信息分享出去</view>
				</view>
			</view>
			<!-- #endif -->
			<view class="cu-dialog">
				<view class="cu-bar bg-white">
					<view class="action">分享到...</view>
					<view class="action text-blue" @tap="doclose">取消</view>
				</view>
				<view class="padding-sm text-black">
					<view class="grid text-center" :class="'col-'+sharecol">
						<view class="share-item" @click="doshareinner()" v-if="shareinner">
							<image class="round" mode="aspectFit" :src="logo"></image>
							<view class="padding-xs">会话</view>
						</view>
						<block v-if="outershare">
							<view class="share-item" @click="doshare(index)" v-if="item.switch" v-for="(item, index) in providers" :key="index">
								<view class="share-icon">
									<text class="ifont" :class="'icon-'+index"></text>
								</view>
								<view class="padding-xs">{{item.title}}</view>
							</view>
							<!-- #ifdef APP-PLUS -->
							<view class="share-item" @click="doShareSystem()">
								<view class="text-xxxl bg-grey round">
									<text class="cuIcon-more"></text>
								</view>
								<view class="padding-xs">更多分享</view>
							</view>
							<!-- #endif -->
							<view class="share-item" @click="docopy(shareinfo.url)" v-if="shareinfo.url!='' && sharecopy">
								<view class="text-xxxl bg-yellow round">
									<text class="cuIcon-copy text-white"></text>
								</view>
								<view class="padding-xs">复制链接</view>
							</view>
						</block>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	export default {
		name: 'share',
		data(){
			return {
				debug:true,
				inwechat:core.inwechat,
				logo:core.system.logo,
				shareresult:'fail',
				theme:core.style,
				batemode:false,
				outershare:core.system.outershare,
				platform:core.platform,
				sharecol:2,
				providers:{
					"weixin":{
						switch:false,
						title:'微信好友'
					},
					"pengyouquan":{
						switch:false,
						title:'朋友圈'
					},
					"qq":{
						switch:false,
						title:'QQ好友'
					},
					"sina":{
						switch:false,
						title:'微博'
					}
				}
			}
		},
		created() {
			if(!this.sharecopy){
				this.sharecol -= 1;
			}
			if(!this.shareinner){
				this.sharecol -= 1;
			}
			let self = this;
			// #ifndef H5
			uni.getProvider({
			    service: 'share',
			    success: function (res) {
					for(let i in res.provider){
						let provider = res.provider[i];
						if(provider=='sinaweibo') provider = 'sina';
						self.providers[provider].switch = true;
						self.sharecol += 1;
						if(provider=='weixin'){
							self.providers.pengyouquan.switch = true;
							self.sharecol += 1;
						}
					}
			    }
			});
			// #endif
		},
		mounted(){
			this.sharecol = this.sharecol>4 ? 5 : 4;
		},
		props: {
			showshare:{
				type:Boolean,
				default:false
			},
			shareinfo : {
				type:Object,
				default:function(){
					return {
						title:'',
						summary:'',
						url:'',
						cover:''
					}
				}
			},
			shareinner : {
				type:Boolean,
				default:true
			},
			sharecopy : {
				type:Boolean,
				default:true
			}
		},
		methods:{
			doclose(){
				this.$emit('closeshare',this.shareresult);
			},
			doshareinner(){
				let self = this;
				return core.post('dialog/shareweb',function(res){
					if(res.type!='success') return core.report(res);
					self.shareresult = 'inner';
					core.navito('dialog/share',{mid:res.message.id,fp:'web'});
				},{title:this.shareinfo.title,url:this.shareinfo.url,cover:this.shareinfo.cover,desc:this.shareinfo.summary});
			},
			doshare(provider){
				if(provider=='') return false;
				let scene = provider=='pengyouquan' ? 'WXSenceTimeline' : 'WXSceneSession';
				let _provider = provider=='pengyouquan'?'weixin':provider;
				let sharetype = provider == 'qq' ? 1 : 0;
				let that = this;
				let summary = this.shareinfo.summary;
				if(provider == 'qq') summary = this.shareinfo.title + ':' + summary + ' 详情请戳：'+ this.shareinfo.url;
				uni.share({
				    provider: _provider,
				    scene: scene,
				    type: sharetype,
					title:this.shareinfo.title,
					imageUrl:this.shareinfo.cover,
					href:this.shareinfo.url,
				    summary: summary,
				    success: function (res) {
						core.toast('分享成功！','','success');
						that.shareresult = 'success';
						that.doclose();
				    },
				    fail: function (err) {
				        console.log("fail:" + JSON.stringify(err));
						that.shareresult = 'fail';
						core.toast('分享失败');
				    }
				});
			},
			docopy(text){
				this.shareresult = 'copy';
				return core.copy(text+"");
			}
			//#ifdef APP-PLUS
			,
			doShareSystem(){
				let self = this;
				let shareType = 'text';
				if(this.shareinfo.cover!="" && (this.shareinfo.title=='分享图片' || this.shareinfo.title=='')){
					shareType = 'image';
				}
				let summary = "#"+ this.shareinfo.title +"# " + this.shareinfo.summary;
				uni.shareWithSystem({
					type:shareType,
					href:this.shareinfo.url,
					imageUrl:this.shareinfo.cover,
					success(res) {
						core.toast('分享成功！','','success');
						self.shareresult = 'success';
						self.doclose();
					}
				});
			}
			//#endif
		}
	}
</script>

<style>
	.sharemodal{z-index: 999; background: rgba(0, 0, 0, 0.8);}
	.sharemodal .bg-arrow{position: absolute; right: 0; top: 0; width: 188upx; height: 172upx; background: url(@/static/images/bg_arrow_topright.png) no-repeat center; background-size: contain;}
	.share-wxapp .bg-arrow{top: calc(var(--status-bar-height) + 120upx); right: 150upx;}
	.sharemodal .center{vertical-align: middle; display: inline-block; text-align: left; line-height: 70upx;}
	.share-wechat{position: fixed; z-index: 999; padding-top: 200upx; top: 0;}
	.share-item image, .share-icon{width: 120upx; height: 120upx;}
	.share-item .text-xxxl{width: 120upx; height: 120upx; line-height: 120upx; margin: 0 auto; margin-bottom: 4upx;}
	.share-icon{text-align: center; line-height: 120upx; margin: 0 auto; margin-bottom: 6upx;}
	.ifont{font-size: 120upx;}
</style>
