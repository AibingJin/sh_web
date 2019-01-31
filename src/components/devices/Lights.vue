<template>
	<div id="light" class="light">
		<!-- <p>dengguang{{room_id}}</p> -->
		<ul>
			<li v-for="(item,index) in lightslist">
				<p>{{item.name}}</p>
				<img :src="item.url" v-if="item.type==='hue'" :id="item.did" :name="item.name" title="点击控制彩灯" @click="hueClick($event)">
				<img v-else :src="item.url">
				<div class="switch switch-mini">
					<button><img :src="openurl" title="打开" :id="item.did" @click="openClick($event)"></button><button><img :src="closeurl" title="关闭" :id="item.did" @click="closeClick($event)"></button>
				</div>
				<!--<div class="switch switch-mini" v-if="item.status=='on'">
                    <input type="checkbox" :id="item.did" :value="item.status" :name="item.type" :checked="true" title="开关" @click="lightClick($event)"/>
                </div>
                <div class="switch switch-mini" v-else>
                    <input type="checkbox" :id="item.did" :value="item.status" :name="item.type" title="开关" @click="lightClick($event)"/>
                </div>-->
			</li>
			<!-- <p>{{switchStatus}}</p> -->
		</ul>
		<v-multilights v-if="mlightFlag" :mulLightsdid="mulLightsdid" :mulLightsname="mulLightsname" v-on:hide="hide"></v-multilights>
	</div>
</template>

<script>
	import { mapGetters } from 'vuex'
	import 'bootstrap-switch/dist/css/bootstrap3/bootstrap-switch.min.css'
	import 'bootstrap-switch/dist/js/bootstrap-switch.min.js'
	import Multilights from '../devices/Multilights'
	export default {
		data() {
			return {
				lightslist: '',
				onurl: require('../../../static/images/light_on.png'),
				offurl: require('../../../static/images/light_off.png'),
				openurl: require('../../../static/images/open-light.png'),
				closeurl: require('../../../static/images/close-light.png'),
				mlightFlag: false,
				mulLightsdid: '',
				mulLightsname: ''
			}
		},
		components: {
			'v-multilights': Multilights
		},
		created() {

		},
		mounted() {
			this.init()
		},
		computed: {
			...mapGetters({
				room_id: 'id',
				ctrltype: 'type',
				switchStatus: 'switchStatus'
			})
		},
		methods: {
			lightClick(e) {
				var that = this;
				var ele = e.target;
				var did = e.target.getAttribute('id');
				var state = e.target.checked ? "on" : "off";
				var key = e.target.getAttribute('name');
				$.post('/run/light/lightControl', {
					did: did,
					key: 'io',
					state: state
				}, function(data) {
					that.lightslist.forEach(function(item, index, arr) {
						if(item.did === data.light.did) {
							item.status = data.light.status;
							item.url = data.light.status === 'on' ? that.onurl : that.offurl
						}
					})
				})
			},
			openClick(e) {
				var ele = e.target;
				var did = e.target.getAttribute('id');
				var state = "on";
				$.post('/run/light/lightControl', {
					did: did,
					key: 'io',
					state: state
				}, function(data) {})
			},
			closeClick(e) {
				var ele = e.target;
				var did = e.target.getAttribute('id');
				var state = "off";
				$.post('/run/light/lightControl', {
					did: did,
					key: 'io',
					state: state
				}, function(data) {})
			},
			hueClick(e) {
				// if(this.mlightFlag){
				// 	this.mlightFlag=false
				// }else{
				this.mlightFlag = true;
				var ele = e.target;
				this.mulLightsdid = e.target.getAttribute('id');
				this.mulLightsname = ele.getAttribute('name');
				// console.log(this.mulLightsname);

				// }
			},
			init() {
				var that = this;
				$.post('/run/light/getLightStates', {
					building_id: sessionStorage.buildID,
					room_id: this.room_id
				}, function(data) {
					data.lights.forEach(function(item, index, arr) {
						if(item.status === 'on') {
							item.url = that.onurl;
						} else {
							item.url = that.offurl;
						}
					});
					that.lightslist = data.lights;
				})
			},
			hide() {
				this.mlightFlag = false;
			}
		},
		watch: {
			room_id: function() {
				this.init();
			},
			switchStatus: function() {
				var that = this;
				$.post(this.ctrltype, {
					did: "all-" + sessionStorage.buildID + "-" + this.room_id,
					key: "io",
					state: this.switchStatus.match(/[a-z]+/g)[0]
				}, function(data) {
					that.lightslist.forEach(function(item, index, arr) {
						item.status = data.lights[index].status;
						item.url = data.lights[index].status === 'on' ? that.onurl : that.offurl
					})
				})
			}
		}
	}
</script>
<style scoped>
	.light {
		position: relative;
		left: 213px;
		top: 100px;
		height: 100%;
		overflow: auto;
	}
	
	.light ul {
		text-align: left;
		/*list-style: none;*/
	}
	
	.light ul li {
		display: inline-block;
		width: 100px;
		margin-left: 60px;
		margin-bottom: 30px;
		text-align: center;
	}
	
	.light .switch button {
		display: inline-block;
		width: 20px;
		height: 20px;
		margin-left: 3px;
		margin-right: 5px;
		vertical-align: middle;
		color: white;
		background: rgba(255, 255, 255, 0);
		border: 0px #3079ED solid;
	}
	
	.light ul li input,
	.light ul img {
		cursor: pointer;
	}
	
	.light ul img {
		margin-bottom: 10px;
	}
	
	.light ul p {
		font-size: 16px;
		font-family: "微软雅黑";
	}
</style>