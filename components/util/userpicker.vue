<template>
	<view class="userpicker">
		<view class="cu-bar search bg-white" v-if="Search">
			<view class="search-form round text-left">
				<text class="cuIcon-search"></text>
				<input :adjust-position="false" type="text" v-model="keyword" placeholder="输入昵称搜索" @input="doSearchUser" @confirm="doSearchUser" confirm-type="search"></input>
				<text class="cuIcon-roundclose text-red" v-if="keyword!=''" @click="doSearchUser('')"></text>
			</view>
		</view>
		<scroll-view :class="pStyle" scroll-y show-scrollbar>
			<view class="cu-list menu text-left" v-if="Mode=='list'">
				<view v-for="(item,index) in members" class="cu-item arrow" :key="index" @longpress.stop="doLongTap(index)" @click="doTapItem(index)">
					<view class="content">
						<text :class="item.style">{{item.nickname}}</text>
					</view>
					<view class="action">
						<view class="cu-tag round light" :class="item.extra" v-if="item.title!=''">{{item.title}}</view>
					</view>
				</view>
			</view>
			<view class="cu-list grid col-5 no-border" v-else>
				<view @longpress.stop="doLongTap(index)" @click="doTapItem(index)" class="cu-item align-center" v-for="(item,index) in members" :key="index">
					<view class="cu-avatar radius lg" :style="'background-image:url('+item.avatar+');'">
						<view class="cu-tag badge" :class="[item.badge, item.extra]" v-if="item.badge!=''"></view>
					</view>
					<text class="text-cut text-sm" :class="item.style">{{item.nickname}}</text>
				</view>
				<view class="cu-item align-center" v-if="AddUrl!=''" @click="naviTo(AddUrl)">
					<view class="dashed">
						<text class="cuIcon-add text-gray"></text>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	export default {
		name: 'userpicker',
		data() {
			return {
				members:[],
				keyword:""
			}
		},
		mounted() {
			
		},
		watch:{
			Users(){
				this.keyword = "";
				this.doSelectUser("");
			}
		},
		props: {
			Search:{
				type: [Boolean, String],
				default: true
			},
			Mode:{
				type: String,
				default: "list"
			},
			AddUrl:{
				type: String,
				default: ""
			},
			pStyle:{
				type: String,
				default: ""
			},
			Users:{
				type:Array,
				default:function(){
					return [];
				}
			}
		},
		methods:{
			doTapItem(index){
				this.$emit('onResult',this.members[index]);
			},
			doLongTap(index){
				this.$emit('onLongtap',this.members[index]);
			},
			doSearchUser(e){
				if(e==""){
					this.keyword = "";
				}
				return this.doSelectUser(this.keyword);
			},
			naviTo(page,data={}){
				return core.navito(page,data);
			},
			doSelectUser(keyword){
				let users = this.Users;
				if(keyword==''){
					return this.members = users;
				}else{
					this.members = [];
				}
				keyword += "";
				for(let i in users){
					let nickname = users[i].nickname + "";
					if(nickname.indexOf(keyword)>=0){
						this.members.push(users[i]);
					}
				}
			}
		}
	}
</script>

<style>
	.userscorller{height: 600upx;}
	.grid .cu-item .dashed{border:4upx dashed #AAAAAA; width: 112upx; height: 112upx; text-align: center; font-size: 36upx; border-radius: 6upx;}
	.grid .cu-item .dashed [class*="cuIcon-"]{margin-top: 0; line-height: 104upx !important;}
	.cu-list.grid>.cu-item .cu-avatar{position: relative;}
	.cu-list.grid>.cu-item .cu-avatar .cu-tag{width: 32upx; font-size: 20upx; margin-top: 0; margin-left: 0; top: -10upx; right: -10upx; left: auto; position: absolute; padding: 0; line-height: 32upx;}
</style>
