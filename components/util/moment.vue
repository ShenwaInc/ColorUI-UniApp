<template>
	<view class="cu-item shadow moment-item" @click="naviTo(item.redirect)">
		<view class="cu-list menu-avatar">
			<view class="cu-item">
				<view class="cu-avatar round lg" @tap.stop="naviTo('member/index',{uid:item.uid})" :style="'background-image:url('+item.avatar+');'"></view>
				<view class="content flex-sub">
					<view>{{item.nickname}}</view>
					<view class="text-sm flex justify-between">
						<text class="text-gray">{{item.datetime}}</text>
						<text class="text-orange" v-if="item.status==0">审核中</text>
					</view>
				</view>
			</view>
		</view>
		<view class="text-content richtext margin-top-sm" v-if="item.content!=''" @tap.stop="naviTo(item.url)">
			<rich-text :nodes="item.content+''"></rich-text>
		</view>
		<view @tap.stop="naviTo('player/index',{vid:item.vid})" class="chat-video padding-lr margin-top-sm" v-if="item.vid>0">
			<view class="bg-img only-img bg-video">
				<image :src="item.poster+(poster>0?'?reload='+poster:'')" mode="widthFix" @error="doReloadPost()"></image>
			</view>
		</view>
		<view class="grid flex-sub padding-lr margin-top-sm" :class="item.extraclass" v-if="item.pics.length>0">
			<view class="bg-img" @tap.stop="doPreviewImage(pic)" :class="item.pics.length==1?'onlyimg':''" v-for="(pic,key) in item.pics" :key="key">
				<image :src="pic" mode="aspectFill"></image>
			</view>
		</view>
		<view class="flex justify-between padding">
			<view :class="'text-'+theme.actcolor">
				<block v-if="item.position.status==1">
					<text class="cuIcon-locationfill"></text>
					<text>{{item.position.name}}({{item.position.distance}})</text>
				</block>
			</view>
			<view class="text-gray text-lg flex">
				<view @tap.stop="doPraise(item.id,index)">
					<text class="cuIcon-appreciatefill margin-lr-xs" :class="item.praised?'text-'+theme.actcolor:''"></text> {{item.praise}}
				</view>
				<view>
					<text class="cuIcon-messagefill margin-lr-xs"></text> {{item.comment}}
				</view>
			</view>
		</view>
		<view class="cu-list menu-avatar comment solid-top" v-if="item.comments.id>0">
			<view class="cu-item noaf">
				<view class="cu-avatar round" @tap.stop="naviTo('member/index',{uid:item.comments.uid})" :style="'background-image:url('+item.comments.avatar+');'"></view>
				<view class="content">
					<view class="text-grey">{{item.comments.nickname}}</view>
					<view class="text-content text-df">
						{{item.comments.content}}
					</view>
					<view class="margin-top-sm flex justify-between">
						<view class="text-gray text-df">{{item.comments.datetime}}</view>
						<view>
							<text class="cuIcon-appreciatefill" :class="item.comments.praised?'text-'+theme.actcolor:'text-gray'"></text>
							<text class="cuIcon-messagefill text-gray margin-left-sm"></text>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="padding-bottom-sm text-center" v-if="item.comment>1">
			<text :style="'color: '+theme.link">查看更多{{item.comment}}条评论</text>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	export default {
		name: 'moment',
		data() {
			return {
				poster:0,
				theme:core.style
			}
		},
		methods:{
			naviTo(page,data={}){
				return core.navito(page,data);
			},
			doReloadPost(){
				if(this.poster==0) return this.poster=1;
				if(this.poster>2) return false;
				let self = this;
				setTimeout(function(){
					self.poster += 1;
				}, 5000)
			},
			doPreviewImage(pic){
				uni.previewImage({
					current:pic,
					urls:this.item.pics
				})
			},
			doPraise(){
				let that = this;
				core.post("album/praise",function(res){
					if(res.type=='success'){
						res.method = 'praise';
						res.index = that.index;
						res.extra = that.extra;
						return that.$emit('onResult',res);
					}else{
						core.toast(res.message,res.redirect,res.type);
					}
				},{aid:this.item.id});
			}
		},
		props:{
			index:{
				type: Number,
				default:0
			},
			extra:{
				type: String,
				default:""
			},
			item:{
				type: Object,
				default:function(){
					return {
						id:0,
						avatar:"",
						comment:0,
						comments:{
							praised:false,
							datetime:"",
							content:"",
							nickname:"",
							avatar:"",
							id:0,
							uid:0
						},
						content:"",
						datetime:"",
						extraclass:"",
						nickname:"",
						pics:[],
						position:{
							address:"",
							distance:"",
							name:"",
							status:0
						},
						praise:0,
						praised:false,
						status:0,
						uid:0,
						url:"",
						vid:0
					}
				}
			}
		}
	}
</script>

<style>
	.cu-item>.text-content{font-size: 32upx !important;}
	.bg-video image{width: 100%;}
	.chat-video .only-img{max-height: 320upx;}
</style>
