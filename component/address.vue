<template>
  <div class="content">
   	<mt-picker :slots="addressSlots" v-if="showAddressType == 1" class="picker" @change="onAddressChange" :visible-item-count="5" ></mt-picker >
   	<mt-picker :slots="streetSlots" v-if="showAddressType == 2" ref="picker" class="picker" @change="onStreetChange" :visible-item-count="5" ></mt-picker >
  </div>
</template>

<script type="text/javascript">

  import Vue from 'vue';
	import {Picker} from 'mint-ui';
	import 'mint-ui/lib/style.css';
	Vue.use(Picker);
	Vue.component(Picker.name, Picker);
	
	export default {
		name : 'address',
		data () {
      return {
        addressSlots: [
          {
            flex: 1,
            defaultIndex: 1,
            values: {},
            className: 'slot1',
            textAlign: 'center'
          }, {
            divider: true,
            content: '-',
            className: 'slot2'
          }, {
            flex: 1,
            values: [],
            className: 'slot3',
            textAlign: 'center'
          }, {
            divider: true,
            content: '-',
            className: 'slot4'
          }, {
            flex: 1,
            values: [],
            className: 'slot5',
            textAlign: 'center'
          }
        ],
        streetSlots: [
          {
            flex: 1,
            values: [],
            className: 'slot1',
            textAlign: 'center'
          }
        ],
        zoneObj: {},
        addressInfo : {
        	province: '',
	        city: '',
	        zone: '',
	        street: ''
        },
        addressZone:''
      }
    },
    beforeCreate: function(){
    	
    	let that = this
		  require.ensure([], function(require){
		  	window.s = require('../json/address4.json');
		  	that.addressSlots[0].values = Object.keys(s);
		  },'clipDoll/addressJson');
    },
    props:[
    	'showAddressType'
    ], //父组件向子组件传递显示哪个面板
    updated: function () {

    	//必须写到这里在数据改变才能触发
		  this.$nextTick(function () {
		    setTimeout(() => {
        	//默认显示
        	this.addressSlots[0].defaultIndex = 0;
        }, 10);
		  })
		},
		watch: {
      'addressZone': {
        handler(val, oval){
          let street = this.zoneObj[this.addressZone]
          this.streetSlots[0].values = street
        }
      }
    },
    methods:{
    	onAddressChange(picker, values) {

    		//添加省-市-县
        let sheng = Object.keys(s);   
        let shi = Object.keys(s[values[0]]); 
        let index = shi.indexOf(values[1]);
        let zone = s[values[0]][shi[index]]; //区
  
        //市
        picker.setSlotValues(1, shi);

	      this.addressInfo.province = values[0];
	      let specialProvince = values[0];

	      if(specialProvince == '台湾省' || specialProvince == '香港特别行政区' || specialProvince == '澳门特别行政区'){
	      	this.addressInfo.city = '';
          this.addressInfo.zone = '';
          this.addressZone = '';
	      }else{
	      	this.addressInfo.city = values[1];
          this.addressInfo.zone = values[2];
          this.addressZone = values[2];   	
	      }

        //区
        if(zone){
        	this.zoneObj = zone;
        	picker.setSlotValues(2, Object.keys(zone));
        }else{
        	picker.setSlotValues(2, '');		
        }
	     	
	     	this.getAddressInfo();
	    },
	    onStreetChange(picker, values){
	    	//添加街道
        this.addressInfo.street = values[0];  

        this.getAddressInfo();
      },
      getAddressInfo(){
      	//自定义事件 传递给父组件
      	this.$emit('transferAddressInfo', this.addressInfo);
      }
    }
	}
</script>

<style type="text/css">
  .address-btn{
		background: red;
	  height: 36px;
	  line-height: 36px;
	  color:#377cf7;
	  background: #f6f6f6;
	  overflow: hidden;
  }
  .address-btn .btn-l{
		float: left;
		padding:0 20px;
  	font-size: 0.4rem;
  }
  .address-btn .btn-r{
  	float: right;
  	padding:0 20px;
  	font-size: 0.4rem;
  }
	.mint-popup{
		width: 100% !important;
		background: #ededed!important;
	}
	.mint-popup .picker-item{
		font-size: 0.38888889rem !important;
	}
	.picker-center-highlight{
		border-top:1px solid #ddd !important;
		border-bottom:1px solid #ddd !important;
	}
</style>