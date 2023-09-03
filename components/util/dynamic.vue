<template>
	<view class="page-moment">
		<swaload :hasTopbar="true" :hasToolbar="!inwechat" v-if="moments.status==0" bgColor="gray"></swaload>
		<block v-else>
			<view v-if="moments.cur==''" class="text-empty">
				<text>{{moments.status==1?'加载中..':'空空如也'}}</text>
			</view>
			<block v-else>
				<scroll-view scroll-x class="bg-white nav special">
					<view @tap="MomentTap(index)" class="cu-item" :class="nav.id==moments.cur?'cur text-'+theme.actcolor:''" v-for="(nav, index) in moments.navs" :key="index">
						<text>{{nav.title}}</text>
					</view>
				</scroll-view>
				<block v-if="moments.cur=='square'">
					<view class="padding-lr padding-top" v-if="userinfo.uid>0">
						<view class="bg-white flex radius-lg" :class="moments.data.square.tags.length==0?'cu-bar search':'padding-sm'">
							<block v-if="moments.data.square.tags.length>0">
								<view class="square-tags">
									<scroll-view scroll-x class="nav xs">
										<view class="cu-item" @click="MomentTag('')" :class="moments.picker.tag==''?'text-olive cur':''">全部</view>
										<view class="cu-item" :class="moments.picker.tag==item?'text-olive cur':''" @click="MomentTag(item)" v-for="(item, index) in moments.data.square.tags" :key="index">
											<text>{{item}}</text>
										</view>
									</scroll-view>
								</view>
								<view class="flex-sub text-lg">
									<text @click="naviTo('album/index', {sm:'citypicker'})" class="cuIcon-locationfill" :class="'text-'+theme.actcolor" v-if="moments.data.square.location"></text>
									<text @click="naviTo('album/post')" :class="moments.data.square.location?'margin-left-sm':''" class="cuIcon-cameraadd text-red" v-if="switchs.albumpost"></text>
									<text @click="naviTo('search/index',{type:'album'})" :class="switchs.search?'margin-left-sm':''" class="cuIcon-search" v-if="switchs.search"></text>
								</view>
							</block>
							<block v-else>
								<view class="cu-avatar round" :style="'background-image:url('+userinfo.avatar+')'"></view>
								<view class="search-form round" @tap="naviTo('album/post')">
									<text class="cuIcon-camerafill"></text>
									<text class="text-gray">今日风和日丽，来说点什么吧~</text>
								</view>
								<view class="action">
									<text @click="naviTo('album/index', {sm:'citypicker'})" class="cuIcon-locationfill" :class="'text-'+theme.actcolor" v-if="moments.data.square.location"></text>
									<text class="cuIcon-search" @click="naviTo('search/index',{type:'album'})"></text>
								</view>
							</block>
						</view>
					</view>
					<view class="cu-card dynamic" v-if="moments.data.square.list.length>0">
						<moment :index="index" @onResult="MomentEmit" :item="item" v-for="(item, index) in moments.data.square.list" extra="square" :key="index"></moment>
					</view>
					<view class="text-center padding" @click="MomentFetch('square', moments.data.square.page+1)" v-if="moments.data.square.loadmore">
						<text class="text-blue">查看更多</text>
					</view>
					<view class="text-empty" v-else><text>{{moments.data.square.list.length>0?'没有更多了~':'暂无数据'}}</text></view>
				</block>
				<block v-else-if="moments.cur=='albums'">
					<view class="cu-card dynamic">
						<moment :index="index" @onResult="MomentEmit" :item="item" v-for="(item, index) in moments.data.albums.list" extra="albums" :key="index"></moment>
					</view>
					<view class="text-center padding" @click="MomentFetch('albums', moments.data.albums.page+1)" v-if="moments.data.albums.loadmore">
						<text class="text-blue">查看更多</text>
					</view>
					<view class="text-empty" v-else>
						<text v-if="userinfo.uid>0">{{moments.data.albums.list.length>0?'没有更多了~':'暂无数据'}}</text>
						<text v-else @click="naviTo('auth/index')" class="text-blue">请先登录</text>
					</view>
				</block>
				<block v-else-if="moments.cur=='nears'">
					<view class="near-page">
						<nears @editinfo="naviTo('ranking/profile')" @showshare="doShowShare()" :autoInit="autoRefresh" :heightextra="140" navstyle="bg-white solid-top"></nears>
					</view>
				</block>
				<block v-else-if="moments.cur=='groups'">
					<view class="cu-list menu-avatar solid-top">
						<view class="cu-item arrow" @click="naviTo('group/detail',{gid:item.id})" v-for="(item, index) in moments.data.groups.list" :key="index">
							<view class="cu-avatar round lg" :style="'background-image:url('+item.pic+');'"></view>
							<view class="content">
								<view class="text-xl">
									<view class="text-cut">{{item.name}}</view>
								</view>
								<view class="text-gray text-sm flex">
									<view class="text-cut">
										{{item.description || '群主是相当的懒，啥都不肯说'}}
									</view>
								</view>
							</view>
						</view>
					</view>
					<view class="text-center padding" @click="MomentFetch('groups', moments.data.groups.page+1)" v-if="moments.data.groups.loadmore">
						<text class="text-blue">查看更多</text>
					</view>
					<view class="text-empty" v-else><text>{{moments.data.groups.list.length>0?'没有更多了~':'暂无数据'}}</text></view>
				</block>
				<block v-else-if="moments.cur=='discovery'">
					<discovery :advsdata="advs.discovery" :switchs="switchs" :navigation="navigation.more"></discovery>
				</block>
			</block>
		</block>
	</view>
</template>

<script>
	import core from "@/core.js"
	import moment from "@/components/util/moment.vue"
	import discovery from "@/components/util/discovery.vue"
	import nears from "@/components/util/nears.vue"
	
	export default {
		name: 'dynamic',
		components: {moment, discovery, nears},
		data() {
			return {
				advs:{
					'maintop':[],
					'splash':[],
					'discovery':[]
				},
				inwechat:core.inwechat,
				theme:core.style,
				userinfo:core.userinfo,
				moments:{
					status:0,
					cur:"",
					curtag:"全部",
					navs:[],
					picker:{
						tag:""
					},
					data:{}
				},
				autoRefresh:0
			}
		},
		mounted(){
			this.MomentInit();
		},
		watch:{
			refresh(){
				this.MomentInit(true);
			}
		},
		props:{
			switchs:{
				type: Object,
				default:function(){
					return {
						albumpost:false,
						search:true
					}
				}
			},
			navigation:{
				type: Object,
				default:function(){
					return {}
				}
			},
			refresh:{
				type:Number,
				default:0
			}
		},
		methods:{
			naviTo(page,data={}){
				return core.navito(page,data);
			},
			doShowShare(){
				this.$emit('showshare');
			},
			MomentEmit(res){
				let index = res.index;
				if(res.method=='praise'){
					if(res.extra=='square'){
						this.moments.data.square.list[index].praised = res.action=='cancel' ? false : true;
						this.moments.data.square.list[index].praise = res.html;
					}else{
						this.moments.data.albums.list[index].praised = res.action=='cancel' ? false : true;
						this.moments.data.albums.list[index].praise = res.html;
					}
				}
			},
			MomentInit(refresh=false){
				if(refresh){
					this.moments.status = 0;
					if(this.moments.cur=='nears'){
						return this.autoRefresh += 1;
					}
				}
				if(this.moments.status>0) return false;
				this.moments.status = 1;
				let self = this;
				core.get('moment',function(res){
					if(typeof(res.type)!='undefined'){
						self.moments.status = 0;
						return core.report(res);
					}
					self.moments.picker = {tag:""};
					self.moments.status = 2;
					self.moments.navs = res.navs;
					self.moments.data = res.data;
					if(res.navs.length>0 && self.moments.cur==''){
						self.MomentTap(0);
					}
				});
			},
			MomentTap(index){
				let nav = this.moments.navs[index];
				if(nav.url!=''){
					return core.navito(nav.url);
				}
				if(nav.id=='nears'){
					this.autoRefresh += 1;
				}
				this.moments.cur = nav.id;
			},
			MomentTag(tag=""){
				this.moments.picker.tag = tag;
				return this.MomentFetch("square", 1);
			},
			MomentFetch(fetch,page=1){
				let postdata = this.moments.picker;
				postdata.cid = fetch;
				postdata.page = page;
				let self = this;
				core.post('moment/fetch',function(res){
					if(typeof(res.type)!='undefined') return core.report(res);
					if(res.page==1){
						self.moments.data[fetch] = res;
					}else{
						self.moments.data[fetch].page = res.page;
						self.moments.data[fetch].loadmore = res.loadmore;
						self.moments.data[fetch].list = self.moments.data[fetch].list.concat(res.list);
					}
				},postdata);
			}
		}
	}
</script>

<style>
	.near-page{position: relative;}
	.page-moment{padding-bottom: 30upx; position: relative; padding-top: 112upx;}
	.page-moment .nav.special{position: fixed; top: 0; left: 0; width: 100%; z-index: 999;}
	.square-tags{padding-right: 15upx; overflow: hidden;}
	.square-tags ~ view{line-height: 56upx;}
	.square-tags .cu-item.cur{border-bottom: none;}
	/* #ifdef MP-WEIXIN || APP-PLUS */
	.page-moment{padding-top: calc(105upx + var(--status-bar-height));}
	.page-moment .nav.special{padding-top: var(--status-bar-height);}
	/* #endif */
</style>