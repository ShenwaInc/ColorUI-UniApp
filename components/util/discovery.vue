<template>
	<view class="discovery">
		<view class="cu-list menu margin-top" v-if="switchs.scan || switchs.search">
			<view v-if="switchs.scan" class="cu-item arrow" @click="doscan()">
				<view class="content">
					<text class="cuIcon-scan text-orange" style="font-size: 40upx;"></text>
					<text class="text-lg">扫一扫</text>
				</view>
			</view>
			<view v-if="switchs.search" class="cu-item arrow" @click="naviTo('search/index')">
				<view class="content">
					<text class="cuIcon-search text-green" style="font-size: 40upx;"></text>
					<text class="text-lg">搜一搜</text>
				</view>
			</view>
		</view>
		<advs :extraclass="'margin-tb-sm'" :advs="advsdata"></advs>
		<view class="cu-list menu" :class="platform=='wxapp'?'margin-top-sm':''" v-if="navigation.length>0">
			<view class="cu-item arrow" @click="naviTo(item.url)" v-for="(item, index) in navigation" :key="index">
				<view class="content" :class="item.extra">
					<text :class="item.icon" style="font-size: 40upx;" v-if="item.icon!=''"></text>
					<text class="cuIcon-link text-blue" style="font-size: 40upx;" v-else></text>
					<text class="text-lg">{{item.name}}</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	import advs from "@/components/util/advs.vue"
	
	export default {
		components: {advs},
		name: 'discovery',
		data(){
			return {
				platform:core.platform
			}
		},
		methods:{
			naviTo(page,data={}){
				return core.navito(page,data);
			},
			doscan(){
				return core.scan();
			}
		},
		props:{
			advsdata:{
				type:Array,
				default:function(){
					return [];
				}
			},
			navigation:{
				type:Array,
				default:function(){
					return []
				}
			},
			switchs:{
				type:Object,
				default:function(){
					return {
						scan:true,
						search:true
					}
				}
			}
		}
	}
</script>

<style>
</style>
