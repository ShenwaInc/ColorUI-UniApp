<template>
	<view>
		<view class="n-page" :class="[pagestyle]" :style="'height:'+pageheight+';'">
			<scroll-view class="n-scroller" scroll-y @scrolltolower="onScrollBottom()">
				<view>
					<view class="flex justify-between" :class="navstyle">
						<view class="nav text-xl text-boldm n-nav">
							<view class="cu-item" v-if="occupations.length>0" @click="showmodal='tags'">
								<text>筛选浏览</text>
								<text class="cuIcon-unfold"></text>
							</view>
							<view class="cu-item">
								<picker :range="ordertext" :value="orders.indexOf(selector.order)" @change="doOrderTap">
									{{ordertext[orders.indexOf(selector.order)]}}
									<text class="cuIcon-unfold"></text>
								</picker>
							</view>
							<view class="cu-item">
								<picker :range="genders" :value="selector.gender" @change="doGenderTap">
									{{genders[selector.gender]}}
									<text class="cuIcon-unfold"></text>
								</picker>
							</view>
						</view>
						<view @click="innermenu=true" class="padding-sm">
							<text class="cuIcon-footprint text-xl text-red"></text>
						</view>
					</view>
					<view class="text-empty" v-if="memberList.length==0">
						<text>{{data.hasposition||updateposed?'暂无数据':'正在获取定位...'}}</text>
					</view>
					<block v-else>
						<view class="nears margin-tb">
							<view class="n-item flex" :class="item.cover==''?'near-1':''" @click="naviTo(item.url)" v-for="(item, index) in memberList" :key="index">
								<view class="flex-sub n-cover padding-right" :class="item.extra">
									<image class="radius-lg" :src="item.avatar" mode="aspectFill" v-if="item.cover==''"></image>
									<block v-else>
										<image class="radius-lg" :src="item.cover" mode="aspectFill" @tap.stop="doViewPics(index,true)"></image>
										<view class="n-pics text-white" v-if="item.pics.length>0" @tap.stop="doViewPics(index)">
											<text class="cuIcon-picfill margin-right-xs"></text>
											<text>{{item.pics.length}}</text>
										</view>
									</block>
								</view>
								<view class="n-content flex-quadruple">
									<view class="flex justify-between align-center">
										<view class="n-nick text-xl">
											<text class="text-bold text-xxl text-cut">{{item.nickname}}</text>
											<text class="margin-left-sm" :class="item.gender==1?'text-blue cuIcon-people':'text-pink cuIcon-friendfamous'" v-if="item.gender>0"></text>
										</view>
										<view class="cu-tag round bg-gold" v-if="item.online"><text class="text-white">在线</text></view>
									</view>
									<view class="n-capsule padding-tb-xs" v-if="item.cover!=''">
										<view class="cu-capsule radius">
											<view class="cu-tag" :class="'bg-'+actstyle">
												<text class="cuIcon-roundcheckfill margin-right-xs"></text>
												真实头像
											</view>
										</view>
									</view>
									<view class="n-tags flex flex-wrap align-end">
										<view class="n-tag" v-if="item.occupation!='' || item.age>0">
											<view class="cu-tag light lg" :class="'bg-'+actstyle">{{item.occupation}}<text v-if="item.age>0 && item.occupation">·</text>{{item.age}}岁</view>
										</view>
										<view class="n-tag" v-for="(tag, ti) in item.tags" :key="ti">
											<view class="cu-tag light lg" :class="'bg-'+actstyle">{{tag}}</view>
										</view>
									</view>
								</view>
							</view>
						</view>
						<view class="n-more">
							<view class="padding text-center text-lg" v-if="hasmore">
								<text :class="'text-'+actstyle">上拉加载更多</text>
							</view>
							<view class="text-empty" v-else>
								<text>没有更多了~</text>
							</view>
						</view>
					</block>
				</view>
			</scroll-view>
		</view>
		<view class="cu-modal top-modal" :style="{top:CustomBar+'px'}" :class="showmodal=='tags'?'show':''" v-if="occupations.length>0">
			<view class="cu-modal-bg" @tap="showmodal=''"></view>
			<view class="cu-dialog">
				<view class="cu-bar text-black">
					<view class="action"></view>
					<view class="content"><text class="text-bold">筛选</text></view>
					<view class="action" @click="showmodal=''">
						<text class="cuIcon-close"></text>
					</view>
				</view>
				<scroll-view scroll-x class="bg-white nav solid-bottom padding-lr-sm" scroll-with-animation>
					<view class="cu-item" :class="index==tabcur?'cur text-'+actstyle:''" v-for="(item, index) in occupations" :key="index" @tap="tabcur=index">
						{{item.title}}
					</view>
				</scroll-view>
				<view class="padding-tb-sm bg-white" v-if="occupations[tabcur].tags.length>0">
					<view class="text-center grid col-3 occupations special">
						<view @click="ChooseTag(tag)" v-for="(tag,index) in occupations[tabcur].tags" :key="index" class="padding-lr-sm padding-tb-xs">
							<view class="cu-tag radius lg" :class="selector.tags.indexOf(tag)>=0?'light line-'+actstyle+' bg-'+actstyle:''">
								<text class="text-lg text-cut">{{tag}}</text>
							</view>
						</view>
					</view>
				</view>
				<view class="text-empty bg-white" v-else>
					<text>暂无数据</text>
				</view>
				<view class="cu-bar bg-white tabbar border shop">
					<view class="submit solid-top">
						<view class="action" @tap="showmodal=''">
							<text class="text-lg">取消</text>
						</view>
						<view class="action" @tap="clearData()">
							<text class="text-lg text-red">清空</text>
						</view>
					</view>
					<view class="submit" :class="'bg-'+actstyle" @tap="initData(1, false, true)">
						<text>确定</text>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-modal bottom-modal" :class="innermenu?'show':''">
			<view class="cu-modal-bg" @tap="doHideMenu()"></view>
			<view class="cu-dialog">
				<view class="cu-list grid col-5 no-border n-action nobg">
					<view class="n-action-item">
						<view class="bg-white radius-lg" @tap="doShowShare()">
							<view class="text-xxxl text-green">
								<text class="cuIcon-share"></text>
							</view>
							<view>分享到...</view>
						</view>
						<!-- #ifdef MP-WEIXIN -->
						<button class="wxsharebtn" open-type="share"></button>
						<!-- #endif -->
					</view>
					<view class="n-action-item" @tap="doEditInfo()">
						<view class="bg-white radius-lg">
							<view class="text-xxxl text-blue">
								<text class="cuIcon-edit"></text>
							</view>
							<view>编辑资料</view>
						</view>
					</view>
					<view class="n-action-item" v-if="updateposed || data.hasposition" @tap="doClearExit()">
						<view class="bg-white radius-lg">
							<view class="text-xxxl text-red">
								<text class="cuIcon-exit"></text>
							</view>
							<view>清除退出</view>
						</view>
					</view>
				</view>
				<view class="flex flex-direction padding-bottom padding-lr">
					<button class="cu-btn bg-grey" @tap="doHideMenu()">取消</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	var InterPos = null;
	
	export default {
		name: 'nears',
		data() {
			return {
				showmodal:"",
				datetime:"",
				innermenu:false,
				tabcur:0,
				hasmore:true,
				pageheight:"calc(100vh - 40)",
				updateposed:false,
				ordertext:["在线优先","距离优先","新人优先"],
				genders:["所有人","只看男生","只看女生"],
				orders:["online","distance","new"],
				selector:{
					gender:0,
					order:'distance',
					page:1,
					occupation:'',
					tags:[],
					lat:"",
					lng:""
				},
				memberList:[],
				occupations:[]
			};
		},
		watch:{
			data(){
				this.memberList = this.data.members;
				this.occupations = this.data.occupations;
				this.selector.gender = this.data.viewgender;
				this.hasmore = this.data.members.length<15 ? false : true;
				this.doGetPostion(0);
			},
			heightextra(){
				this.resize();
			},
			tabbar(){
				this.resize();
			},
			showmenu(){
				this.innermenu = this.showmenu;
			},
			autoInit(){
				if(!this.updateposed) return false;
				this.initData(1);
			}
		},
		props: {
			data: {
				type: Object,
				default: function(){
					return {
						hasposition:false,
						interlocation:0,
						members:[],
						occupations:[],
						lbskey:"",
						viewgender:0
					}
				}
			},
			tabbar:{
				type:[Boolean, String],
				default:false
			},
			frompage:{
				type:String,
				default:'home'
			},
			showmenu:{
				type:[Boolean, String],
				default:false
			},
			pagestyle:{
				type:String,
				default:function(){
					return core.style.navbg;
				}
			},
			heightextra:{
				type:Number,
				default:0
			},
			actstyle:{
				type:String,
				default:function(){
					return core.style.actcolor;
				}
			},
			autoInit:{
				type:Number,
				default:0
			},
			navstyle:{
				type:String,
				default:function(){
					return core.style.actcolor;
				}
			}
		},
		mounted(e){
			this.resize();
			this.memberList = this.data.members;
			this.selector.gender = this.data.viewgender;
			this.hasmore = this.data.members.length<15 ? false : true;
			this.doGetPostion(0);
		},
		methods:{
			resize(){
				let Topbar = this.StatusBar;
				let Tabbar = '';
				// #ifdef H5
				if(this.heightextra>0){
					Tabbar = ' - '+this.heightextra+'upx';
				}
				if(!core.inwechat || this.frompage!='home'){
					Topbar += this.CustomBar;
				}
				if(this.tabbar){
					Tabbar += ' - 120upx - env(safe-area-inset-bottom) / 2';
				}
				this.pageheight = "calc(100vh - "+Topbar+"px"+Tabbar+")";
				// #endif
				// #ifndef H5
				if(this.tabbar){
					Tabbar = " - 120upx";
				}
				this.pageheight = "calc("+core.Client.windowHeight+"px"+Tabbar+")";
				// #endif
			},
			naviTo(page,data={}){
				return core.navito(page,data);
			},
			onScrollBottom(){
				if(!this.hasmore) return false;
				let page = this.selector.page + 1;
				return this.initData(page);
			},
			doEditInfo(){
				this.doHideMenu();
				this.$emit('editinfo');
			},
			doShowShare(){
				this.doHideMenu();
				this.$emit('showshare');
			},
			doOrderTap(e){
				this.selector.order = this.orders[e.detail.value];
				this.initData(1);
			},
			doGenderTap(e){
				let gender = e.detail.value;
				this.selector.gender = gender;
				this.initData(1);
			},
			ChooseTag(tag){
				let index = this.selector.tags.indexOf(tag);
				if(index>=0){
					this.selector.tags.splice(index,1);
				}else{
					this.selector.tags.push(tag);
				}
			},
			doViewPics(index,iscover=false){
				let picjsons = JSON.stringify(this.memberList[index].pics);
				let pics = JSON.parse(picjsons);
				if(iscover){
					pics.unshift(this.memberList[index].cover);
				}
				return uni.previewImage({
					current:pics[0],
					urls:pics,
					indicator:'number'
				});
			},
			doUpdatePostion(ret){
				uni.hideLoading();
				if(typeof(ret.lat)=='undefined' || !ret.lng) return false;
				//console.log('获取到位置信息：', ret);
				this.updateposed = true;
				let self = this;
				this.selector.lat = ret.lat;
				this.selector.lng = ret.lng;
				this.$emit("Position", {poslat:ret.lat,poslng:ret.lng});
				return core.post("member/position",function(res){
					if(self.data.interlocation>0){
						if(InterPos==null){
							InterPos = setInterval(function(){
								self.doGetPostion(1);
							}, self.data.interlocation);
						}
					}else if(InterPos!=null){
						clearInterval(InterPos);
						InterPos = null;
					}
					self.selector.page = 1;
					return self.initData(1);
				},{poslat:ret.lat,poslng:ret.lng});
			},
			doGetPostion(times=0){
				if(this.updateposed && times==0) return false;
				uni.showLoading({
					title:"获取定位中..."
				});
				if(times==0 && core.userinfo.uid>0){
					this.initData(1);
				}
				let toast = times==0 ? true : false;
				core.getpos(this.doUpdatePostion, toast);
			},
			doClearExit(){
				let self = this;
				return core.confirm('退出后别人也无法通过附近找到你','清除位置并退出',function(){
					core.post('ranking/nearby',function(res){
						self.showmodal = '';
						if(res.type!='success') return core.report(res);
						core.toast(res.message,res.redirect,'success');
					},{clearexit:"true"})
				},{},function(){
					self.showmodal = '';
				});
			},
			doHideMenu(){
				this.innermenu = false;
				this.$emit('hidemenu');
			},
			clearData(){
				this.selector.tags=[];
				this.initData(1, false, true);
			},
			initData(page=1, callBack, hidemodal=false){
				let postdata = this.selector;
				let self = this;
				postdata.page = page;
				return core.post('ranking/nears',function(res){
					if(typeof(res.type)!='undefined') return core.report(res);
					self.doHideMenu();
					if(hidemodal){
						self.showmodal = '';
					}
					self.selector.page = page;
					if(page==1){
						self.memberList = res.members;
					}else{
						self.memberList = self.memberList.concat(res.members);
					}
					if(typeof(res.occupations)!='undefined'){
						self.occupations = res.occupations;
					}
					if(res.gender!=self.selector.gender){
						self.selector.gender = res.gender;
					}
					self.hasmore = res.members.length<15 ? false : true;
					if(typeof(callBack)=='function'){
						callBack(res);
					}
				},postdata);
			}
		}
	}
</script>

<style>
	.n-page{position: absolute; top: 0; width: 100%;}
	.n-scroller{height: 100%;}
	.nears .n-item{padding: 0 30upx; padding-bottom: 10upx; position: relative;}
	.nears>.n-item:last-child:after{border: none;}
	.bg-dazzledark .nears>.n-item:after{border-color: #545356;}
	.n-item + .n-item{margin-top: 20upx;}
	.n-cover, .viewpic{position: relative;}
	.n-cover image{height: 267upx; width: 182upx; display: block;}
	.near-1 .n-cover image{height: 182upx !important;}
	.n-cover .n-pics{position: absolute; bottom: 0; left: 0upx; padding: 6px 15upx; background-color: rgba(0,0,0,0.2); border-radius: 0 12upx;}
	.bg-dazzledark .n-nav .text-dazzledark.cur{color: #cfa86f;}
	.n-content{overflow: hidden;}
	.n-capsule .bg-dazzledark{background-color: #d29e56; color: #1e1d06;}
	.n-tags{overflow: hidden; white-space: nowrap; text-overflow: ellipsis; margin-left: -10upx; min-height: 120upx;}
	.n-tags .cu-tag{border-radius: 4upx;}
	.n-tags .cu-tag.bg-dazzledark{background-color: #1e1920; color: #b88c52;}
	.n-tag{padding: 6upx;}
	.n-action{padding: 30upx 24upx !important;}
	.n-action-item{padding:0 6upx; display: inline-block !important; position: relative;}
	.n-action-item>view{padding: 15upx 6upx;}
	.n-action-item .wxsharebtn{position: absolute; top: 0; left: 0; width: 100%; height: 100%; opacity: 0;}
	.occupations .cu-tag{width: 100%;}
	.cu-modal .nav .cu-item{margin: 0;}
	.grid.special .cu-tag.lg{height: 70upx; line-height: 70upx;}
	.n-nav.nav .cu-item{margin: 0;}
</style>
