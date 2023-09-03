<template>
	<view>
		<remote-js :src="src" @loadjsfinish="onjsLoaded"></remote-js>
	</view>
</template>

<script>
	export default {
		name: 'remotejs',
		components:{
			'remote-js': {
			  render (createElement) {
			    var self = this
			    return createElement('script', {
			      attrs: { type: 'text/javascript', src: this.src },
			      on: {
			        load: function () {
			          self.$emit('loadjsfinish')
			        }
			      }
			    })
			  },
			  props: {
			    src: { type: String, required: true }
			  }
			}
		},
		props: {
		  src: { type: String, required: true }
		},
		methods:{
			onjsLoaded(e){
				this.$emit('jsloaded',e);
			}
		}
	}
</script>

<style>
</style>
