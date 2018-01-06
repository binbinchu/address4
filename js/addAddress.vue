<template>
  <div class="express-container fadeIn">
    <mt-popup
    v-model="popupVisible"
    position="bottom">
      <div class="address-btn">
        <span class="btn-r" @click="closeAddressLayer">完成</span>
      </div>
      <select-address 
      @transferAddressInfo="getAddressInfo" :showAddressType="addressType"
      ></select-address>
    </mt-popup>
    <span class="bg_mask"></span>
    <div class="express-content">
      <div class="edit-area">
          <div class="name-wrap">
            <label for="">收货人</label>
            <input type="text" v-model="userName">
          </div>
          <div class="mobile-wrap">
            <label for="">手机号码</label>
            <input type="text" v-model="mobile">
          </div>
          <div class="select-address-wrap" @click="showAddressPopup">
            <label for="">所选地区</label>
            <span>{{addressInfo.province}}</span>
            <span>{{addressInfo.city}}</span>
            <span>{{addressInfo.zone}}</span>
            <span class="select-address-btn fr"><span v-if="showAddressType_1">请选择</span><i></i></span>
          </div>
          <div class="select-address-wrap street" @click="showStreetPopup">
            <label for="">街道</label>
            <span>{{addressInfo.street}}</span>
            <span class="select-address-btn fr"><span v-if="showAddressType_2">请选择</span><i></i></span>
          </div>
          <div class="edit-addr-wrap">
            <textarea  placeholder="请填写详细地址，不少于5个字" v-model="address"></textarea>
          </div>
      </div>
      <div class="express-btn-wrap">
        <span @click="saveAddress">保存</span>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  import common from 'common';
  import selectAddress from '../components/address.vue';
  import Vue from 'vue';
  import { Popup } from 'mint-ui';
  import 'mint-ui/lib/style.css';
  import base from '../common/base';
  Vue.component(Popup.name, Popup);
  Vue.use(base);

  export default {
    data () {
      return {
        userName:'',
        mobile:'',
        address:'',         //详细地址
        addressInfo:{},     //子组件传过来的值
        popupVisible:false, //控制子组件popup
        addressType:'0',
        showAddressType_1:true,
        showAddressType_2:true,
        addressJson:{}
      }
    },
    props:[
     // 'showAddressType'
    ],
    mounted:function(){
      document.title = '编辑收货地址';
    },
    methods: {
      //保存接口
      saveAddress(){

        //校验用户填写信息    
        let verifyInputInfoResult = this.verifyInputInfo(this.userName, this.mobile, this.addressInfo, this.address);

        //校验不通过
        if(!verifyInputInfoResult){
          return false;
        }

        //过滤详细地址
        let _address = this.verifyText(this.address);

        //行政区（示例：广东省,深圳市,南山区）
        let _region = this.addressInfo.province + ',' + this.addressInfo.city + ',' + this.addressInfo.zone;  
    
        //写入地址
        this.writeInAddr({
          realname : this.userName,
          mobile : this.mobile,
          address : _address,
          region : _region,
          street : this.addressInfo.street
        })
      },
      writeInAddr(addrObj){
        //添加地址

        let that = this;
        common.showLoading();

        axios.post('/user/addaddr', {
            realname : addrObj.realname,
            mobile :  addrObj.mobile,
            address : addrObj.address,
            region : addrObj.region,
            street : addrObj.street
        }).then((res) => {
          common.hideLoading()
          let data = res.data

          if(data.code == 0){
            //保存成功回到收货地址页面
            that.saveSucess();

          }else{
            layer.open({
              content: data.message,
              skin: 'msg',
              time: 3
            })
          }
        },
        (err) => {
          console.log(err)
        }) 
      },
      saveSucess(){
        let that = this;
        layer.open({
          content:'保存成功',
          skin:'msg',
          time:2,
          end:function(){
            that.$goRoute('/setAddress')
          }
        });
      },
      getAddressInfo(msg){
        //组件传过来的值
        this.addressInfo = msg;
      },
      showAddressPopup(){

        //显示地址组件
        this.popupVisible = true;
        this.addressType = '1';
        this.showAddressType_1 = false;  

      },
      showStreetPopup(){
        //显示街道
        this.popupVisible = true;
        this.addressType = '2';
        this.showAddressType_2 = false;
      },
      closeAddressLayer(){
        this.popupVisible = false;     
      }  
    },
    components: {
      'selectAddress': selectAddress
    }
  }
</script>