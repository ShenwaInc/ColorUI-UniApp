<template>
	<view class="citypicker">
		<view class="cu-modal" :class="ShowPicker?'show':''">
			<view class="cu-modal-bg" @tap="doClose()"></view>
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">{{title||'选择地区'}}</view>
					<view class="action" @tap="doClose()">
						<text class="cuIcon-close text-red"></text>
					</view>
				</view>
				<view class="bg-white">
					<picker :value="ProvinceIndex" :range="provinceList" @change="ChooseProvince">
						<view class="cu-list menu text-left">
							<view class="cu-item arrow">
								<view class="content">
									<text class="text-black">省份</text>
								</view>
								<view class="action">
									<view class="cu-tag round light" :class="'bg-'+theme.actcolor">{{curdata.province||'请选择'}}</view>
								</view>
							</view>
						</view>
					</picker>
					<picker :value="CityIndex" :range="cityAllList[ProvinceIndex]" @change="ChooseCity">
						<view class="cu-list menu text-left">
							<view class="cu-item arrow">
								<view class="content">
									<text class="text-black">城市</text>
								</view>
								<view class="action">
									<view class="cu-tag round light" :class="'bg-'+theme.actcolor">{{curdata.city||'请选择'}}</view>
								</view>
							</view>
						</view>
					</picker>
				</view>
				<view class="cu-bar bg-white justify-end" v-if="Showconfirm">
					<view class="action">
						<button class="cu-btn line-gray text-gray" @tap="doClose()">取消</button>
						<button class="cu-btn line-red text-red margin-left" @tap="doClear()" v-if="showclear">{{textclear}}</button>
						<button class="cu-btn margin-left" :class="'bg-'+theme.actcolor" @tap="doConfirm()">{{textbtn||'确定'}}</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	export default {
		name: 'citypicker',
		data() {
			return {
				address: [],
				AreaList:[],
				cityAllList: [],
				CityIndex:0,
				curdata:{
					province:"",
					city:""
				},
				ProvinceIndex:0,
				provinceList: [],
				showclear:false,
				theme:core.style
			};
		},
		mounted() {
			this.curdata.province = this.province,
			this.curdata.city = this.city;
			let self = this;
			let cachekey = '_xfy_whotalk_address_';
			core.cacheread(cachekey,function(res){
				self.initAddress(res.data);
			},function(e){
				let jsonurl = core.system.siteroot + '/addons/xfy_whotalk/static/address.json';
				uni.request({
					url:jsonurl,
					method:'GET',
					dataType:'json',
					success:function(res){
						core.cacheset(cachekey,res.data);
						self.initAddress(res.data);
					},
					fail:function(e){
						core.toast('地区列表加载失败');
						console.log(e);
					}
				});
			});
		},
		methods:{
			doClose(){
				this.$emit('onCancel',true);
			},
			doClear(){
				this.curdata = {
					province:"",
					city:""
				};
				this.$emit('onClear',true);
			},
			doConfirm(){
				if(this.curdata.province=="") return core.toast("请选择省份");
				if(this.curdata.city=="" && this.PickMode==1) return core.toast("请选择城市");
				this.$emit('onResult',this.curdata);
				if(this.textclear!=''){
					this.showclear = true;
				}
			},
			ChooseProvince(e){
				this.ProvinceIndex = e.detail.value;
				this.curdata.province = this.provinceList[this.ProvinceIndex];
				if(this.PickMode==0 && !this.Showconfirm){
					this.curdata.city = "";
					return this.doConfirm();
				}
				this.CityIndex = 0;
				this.curdata.city = this.cityAllList[this.ProvinceIndex][0];
			},
			ChooseCity(e){
				this.CityIndex = e.detail.value;
				this.curdata.city = this.cityAllList[this.ProvinceIndex][this.CityIndex];
				if(!this.Showconfirm) this.doConfirm();
			},
			initAddress(address){
				if(core.isJsonString(address)){
					address = JSON.parse(address);
				}
				const that = this;
				// 所有城市列表,二维数组
				let cityAllList = [];
				// 省列表
				let provinceList = [];
				// address为省市区的json数据
				//let AreaList = [];
				for(let key in address){
					provinceList.push(address[key].name);
					if(that.province==address[key].name){
						that.ProvinceIndex = key;
					}
					let citylist = [];
					//AreaList[key] = [];
					for(let kee in address[key].children){
						if(that.ProvinceIndex==key && that.city=='' && that.CityIndex==0){
							that.CityIndex = kee;
						}
						citylist.push(address[key].children[kee].name);
						if(that.city==address[key].children[kee].name){
							that.CityIndex = kee;
							if(that.ProvinceIndex==0){
								that.ProvinceIndex = key;
							}
						}
					}
					cityAllList.push(citylist);
				}
				that.provinceList = provinceList;
				that.cityAllList = cityAllList;
				that.address = [provinceList, cityAllList[0]];
				return true;
			}
		},
		props: {
			ShowPicker:{
				type: [Boolean, String],
				default: false
			},
			Showconfirm:{
				type: Boolean,
				default: true
			},
			PickMode:{
				type: Number,
				default: 1	//模式，0仅省份，1省份/城市
			},
			province:{
				type: String,
				default:""
			},
			city:{
				type: String,
				default:""
			},
			title:{
				type: String,
				default:""
			},
			textbtn:{
				type: String,
				default:""
			},
			textclear:{
				type: String,
				default:""
			}
		}
	}
</script>

<style>
</style>
