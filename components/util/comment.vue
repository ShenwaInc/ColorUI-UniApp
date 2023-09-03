<template>
	<view>
		<view class="comments mini padding-top" v-if="comments.length>0">
			<view class="cu-list menu-avatar padding-left-sm" v-for="(item, index) in comments" :key="index">
				<view class="cu-item noaf" @click="naviTo('member/index',{uid:item.uid})">
					<view class="cu-avatar round" :style="'background-image:url('+item.avatar+');'"></view>
					<view class="content">
						<view class="text-black">
							{{item.nickname}}
							<text class="cuIcon-delete text-red margin-left" @tap.stop="removeComment(item.id)" v-if="userId==item.uid"></text>
						</view>
					</view>
					<view class="action">
						<view class="text-grey text-xs">{{item.datetime}}</view>
					</view>
				</view>
				<view class="summary">{{item.content}}</view>
			</view>
		</view>
		<view v-else-if="showempty" class="text-empty">
			<text>空空如也</text>
		</view>
	</view>
</template>

<script>
	import {mapState} from 'vuex'
	import core from "@/core.js"
	
	export default {
		computed: mapState(['hasLogin','userId']),
		name: 'comment',
		props: {
			showempty:{
				type:Boolean,
				default:false
			},
			comments:{
				type:Array,
				default:function(){
					return [];
				}
			}
		},methods:{
			naviTo(page,data){
				return core.navito(page,data);
			},
			removeComment(cid){
				var that = this;
				return core.confirm(core.langs.text_delete_confirm,core.langs.text_confirm_title,function(){
					core.get('comment/delete',function(res){
						if(res.type!='success') return core.report(res);
						that.reloadComment();
					},{cid:cid});
				});
			},
			reloadComment(){
				this.$emit('reload');
			}
		}
	}
</script>

<style>
	.comments .cu-item{height: 110upx !important;}
	.comments .cu-item>.action{width: 150upx !important;}
	.comments .summary{padding-left: 150upx; color: #999999; font-size:26upx; font-family:PingFangSC-Regular,PingFang SC; line-height: 40upx; padding-right: 20upx;}
	.comments.mini .cu-item{-webkit-align-items: inherit; align-items: inherit; height: 50upx !important;}
	.comments.mini .cu-item .content{left: 120upx;}
	.comments.mini .summary{font-size: 0.69rem; line-height: 1.2rem; padding-left: 120upx; padding-right: 20upx; text-align: justify; padding-bottom: 20upx; position: relative;}
	.comments.mini .summary::after{position: absolute; content: ''; width: 100%; height: 1upx; left: 0; bottom: 0; background: #F7F7F7;}
	.comments.mini .cu-list:last-child .summary::after{display: none;}
</style>
