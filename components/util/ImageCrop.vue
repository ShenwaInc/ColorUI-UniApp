<template>
	<view class="cu-modal show" v-if="sCroping">
		<view class="cu-dialog crop-container no-radius">
			<bt-cropper ref="cropper" :ratio="1" :dWidth="cropWidth" :imageSrc="cropImage">
				<view class="cu-bar bg-white tabbar shop">
					<view class="submit" @click="sCroping=false">
						<text>取消</text>
					</view>
					<view class="submit bg-orange" @click="doImageSave()" v-if="isSquare">
						<text>原图</text>
					</view>
					<view class="submit bg-green" @tap="doImageCropper()">裁切</view>
				</view>
			</bt-cropper>
		</view>
	</view>
</template>

<script>
	import core from "@/core.js"
	
	export default {
		name: 'ImageCrop',
		data(){
			return {
				sCroping:false
			}
		},
		watch:{
			cropImage(){
				if(this.cropImage!=""){
					this.sCroping = true;
				}
			}
		},
		props: {
			cropImage : {
				type:String,
				default:""
			},
			cropTempFile : {
				type:String,
				default:""
			},
			cropWidth:{
				type:Number,
				default:192
			},
			isSquare:{
				type:Boolean,
				default:false
			},
			resFrom:{
				type:String,
				default:""
			}
		},
		methods:{
			doImageSave(){
				let self = this;
				let data = {};
				if(this.resFrom!=""){
					data.resfrom = this.resFrom;
				}
				core.upload(this.cropTempFile,function(res){
					self.$emit('cropRes', res);
					self.sCroping = false;
				}, 'image', 'attach/upload', data);
			},
			doImageCropper(){
				let self = this;
				this.$refs.cropper.crop().then(([err,cropres])=>{
					if(err){
						console.log(err)
						return core.toast('裁剪失败，请重试');
					}
					let callback = function(file){
						core.upload(file,function(res){
							self.$emit('cropRes', res);
							self.sCroping = false;
						});
					}
					// #ifdef H5
					let base64 = cropres.tempFilePath
					let blob = this.h5SaveCanvasImage(base64);
					return uni.getImageInfo({
						src:blob,
						success(Croper) {
							//console.log(Croper);
							let img = new Image();
							img.src = Croper.path;
							let canvas = document.createElement('canvas');
							let ctx = canvas.getContext('2d')
							canvas.width     = Croper.width 
							canvas.height    = Croper.height
							ctx.drawImage(img, 0, 0, Croper.width, Croper.height)
							canvas.toBlob(function(fileSrc) {
								let fileObj = new File([fileSrc],new Date().getTime()+'.jpg',{type:'image/jpeg'});
								callback(fileObj);
							},'image/jpeg',1);
						},
						fail() {
							core.toast('裁剪失败，请重试');
						}
					});
					// #endif
					callback(cropres.tempFilePath);
				});
			},
			h5SaveCanvasImage(base64) {
				// copnsole.log(base64)
				var arr = base64.split(','),
					mime = arr[0].match(/:(.*?);/)[1],
					bstr = atob(arr[1]),
					n = bstr.length,
					u8arr = new Uint8Array(n);
				while (n--) {
					u8arr[n] = bstr.charCodeAt(n);
				}
				let blob = new Blob([u8arr], {
					type: mime
				});
				return URL.createObjectURL(blob);
			},
		}
	}
</script>

<style>
	.crop-container{height: 100vh; width: 100vw;}
</style>