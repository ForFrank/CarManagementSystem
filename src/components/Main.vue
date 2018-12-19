<template>
  <div id="app" v-cloak>
    <el-container id="Main_container">
      <!--头部-->
      <el-header >
        <h2>车辆管理系统</h2>
        <Nowtime></Nowtime>
      </el-header>
      <el-container>
        <!--侧栏-->
        <el-aside width="200px" >
          <el-menu :default-openeds="['1']" background-color="#0d2135">
            <el-submenu index="1">
              <div slot="title">地图信息</div>
              <el-menu-item-group>
                <el-menu-item index="1-1">
                  <div  id="history" @click="showDevice">历史回放</div>
                </el-menu-item>
                <el-menu-item index="1-2">
                  <div id="realtime" @click="toggle()">定位查询</div>
                </el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu index="2">
              <div slot="title" style="">用户信息</div>
              <el-menu-item-group>
                <el-menu-item index="2-1">
                  <a href="#">list1</a>
                </el-menu-item>
                <el-menu-item index="2-2">
                  <a href="#">list2</a>
                </el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu index="3">
              <div slot="title">设备信息</div>
              <el-menu-item-group>
                <el-menu-item index="3-1">
                  <a href="#">list1</a>
                </el-menu-item>
                <el-menu-item index="3-2">
                  <a href="#">list2</a>
                </el-menu-item>
              </el-menu-item-group>
            </el-submenu>
            <el-submenu index="4">
              <div slot="title">车辆信息</div>
              <el-menu-item-group>
                <el-menu-item index="4-1">
                  <a href="#">list1</a>
                </el-menu-item>
                <el-menu-item index="4-2">
                  <a href="#">list2</a>
                </el-menu-item>
              </el-menu-item-group>
            </el-submenu>

          </el-menu>
        </el-aside>
        <!--内容-->
        <el-main id="imp" style="text-align: left; height: 100%;">
          <Map></Map>
          <!--设备列表-->
          <div  v-if="deviceflag"  id="replayUI" >
            <div>
              <p style="width:50%;float:left; font-size: 15px">历史回放</p>
              <button style="float:right;" @click="toggle()" id="buttonclick">关闭</button>
            </div>
            <div class="table-head">
              <table bgcolor="#545557"  border="1" cellpadding="0" cellspacing="0" bordercolor="black">
                <colgroup><col style="width: 80px;" /><col style="width: 100px;"/><col style="width: 130px;"/></colgroup>
                <thead>
                <tr>
                  <td>车辆ID</td>
                  <td>车辆名称</td>
                  <td>日期</td>
                  <td>操作</td>
                </tr>
                </thead>
              </table>
            </div>
            <div class="table-body">
              <table bgcolor="#545557"  border="1" cellpadding="0" cellspacing="0" bordercolor="black">
                <colgroup><col style="width: 80px;" /><col style="width: 100px;"/><col style="width: 130px;"/></colgroup>
                <tbody id="tby">
                <tr v-for="(item,index) in devicelists" :key="index.id" class="first">
                  <td >{{item.deviceCode}}</td>
                  <td >{{item.deviceName}}</td>
                  <td >{{item.creatTime}}</td>
                  <td ><div><el-button type="text" size="mini" :id="item.deviceCode" ref="myButton" @click="replay(item.deviceCode)">回放
                  </el-button></div></td>
                </tr>
                </tbody>
              </table>
            </div>
          </div>
          <!--轨迹列表-->
          <div  v-if="trajectoryflag"  id="trajectoryUI" >
            <div style="height: 25px">
              <p style="width:50%;float:left; font-size: 15px">历史回放</p>
              <button style="float:right;" @click="toggle()" id="trajectorybutton">关闭</button>
            </div>
            <div style="height: 50px; " >
              <table width="100%">
                <tbody>
                  <tr>
                    <td style="text-align: left">开始时间：{{trajectorylists[0].trackTime}}</td>
                    <td style="text-align: center">设备ID：{{trajectorylists[0].deviceCode}}</td>

                  </tr>
                  <tr>
                    <td style="text-align: left">结束时间：{{trajectorylists[trajectorylists.length-1].trackTime}}</td>
                    <td style="text-align: center"><el-button type="text" size="mini" style="float:right;" @click="showDevice" id="trajectoryelbutton">返回</el-button></td>
                  </tr>
                </tbody>



              </table>
              <!--<ul>
                <li style="float: left">开始时间：{{trajectorylists[0].trackTime}}</li>
                <li style="float: right">结束时间：{{trajectorylists[trajectorylists.length-1].trackTime}}</li>
                <li style="float: left">设备ID：{{trajectorylists[0].deviceCode}}</li>
                <li><el-button type="text" size="mini" style="float:right;" @click="showDevice" id="trajectoryelbutton">返回</el-button></li>
              </ul>-->

            </div>
            <div class="table-head">
              <table bgcolor="#545557"  border="1" cellpadding="0" cellspacing="0" bordercolor="black">
                <colgroup><col style="width: 130px;" /><col style="width: 100px;"/><col style="width: 100px;"/><col style="width: 50px;"/></colgroup>
                <thead>
                <tr>
                  <td>轨迹时间</td>
                  <td>经度</td>
                  <td>纬度</td>
                  <td>车速</td>
                </tr>
                </thead>
              </table>
            </div>
            <div class="tra-table-body">
              <table bgcolor="#545557"  border="1" cellpadding="0" cellspacing="0" bordercolor="black">
                <colgroup><col style="width: 130px;" /><col style="width: 100px;"/><col style="width: 100px;"/><col style="width: 50px;"/></colgroup>
                <tbody>
                <tr v-for="(item,index) in trajectorylists" :key="index.id">
                  <td >{{item.trackTime}}</td>
                  <td >{{Math.round((item.longitude)*1000000)/1000000}}</td>
                  <td >{{Math.round((item.latitude)*1000000)/1000000}}</td>
                  <td >{{item.mobileyeChinaVelocity}}</td>
                </tr>
                </tbody>
              </table>
            </div>
          </div>
          <!--<div id="trajectoryUI"></div>-->

        </el-main>
      </el-container>
    </el-container>
  </div>

</template>
<script>
  import Map from '../components/Map.vue'
  import axios from 'axios';
  import $ from 'jquery';
  import bus from '../components/bus.vue'
  import Nowtime from './Nowtime.vue'
  export default {
    name: 'Form',
    components: {Map,Nowtime},
    data () {
      return {
        deviceflag:false,
        trajectoryflag:false,
        devicelists:[],
        trajectorylists:[],
        msg:[],
      };
    },
    computed: {
      user(){
        return this.$store.state.user
      }
    },
    methods:{
      toggle() {
        var _this = this;
        _this.deviceflag=false;
        _this.trajectoryflag=false
      },
      showDevice:function(){
        var _this = this;
          //调用设备查询接口
          var queryAllDeviceUrl=window.g.queryAllDevice_URL;
          axios.post(queryAllDeviceUrl, {}).then(function (response) {
            _this.deviceflag=true;
            _this.trajectoryflag=false;
            _this.devicelists=response['data'].data;
            // console.log( _this.lists);
            // $("body").undelegate();
          }).catch(function(error){
            console.log(error);
          });
      },
      replay:function(event){
        var _this = this;
        console.log( event);
        var msg = [];
        //调用历史轨迹查询接口
        var queryTrackUrl=window.g.queryTrack_URL;
        axios.get(queryTrackUrl,{params:{deviceCode:event}}).then(function (response) {
          _this.deviceflag=false
          _this.trajectoryflag=true;
          _this.trajectorylists=response['data'].data;
          $.each(response['data'].data,function(idx,obj){
            // msg.push(idx,parseFloat(obj.longitude),parseFloat(obj.latitude),0);
            msg.push(obj);
            _this.msg = msg;
          });
          bus.$emit('trajectory',msg);
        }).catch(function(error){
          console.log(error);
        });
        this.$emit('click',event)
      }
    },
    mounted(){

    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    height: 100%;
    /*margin-top: 60px;*/
  }
  div {
    color: white;
  }
  a {
    color: white;
  }
  h2 {
    /*top: 30px;*/
    text-align: center;
    color: white
  }
  #Main_container {
    height: 100%;
    /*margin-top: 0;*/
    /*background-color: white;*/
    background-color: #0d2135;
  }
  #imp {
    position: relative;
  }
  #replayUI{
    position: absolute;
    left:50px;
    top:50px;
    width: 400px;
    height: 420px;
    background: #545557;
    padding: 5px 5px 0;
  }
  .el-table--enable-row-hover .el-table__body tr:hover>td{
    background-color: black ;
  }
  td {
    text-align:center;
    font-size: 13px;
    /*width: 100px;*/
  }
  .table-head{
    padding-right:17px;}
  .table-body{
    width:100%;
    height:360px;
    overflow-y:scroll;
  }
  .table-head table,.table-body table{
    width:100%;}
  .table-head td{
    font-size: 15px;
  }
  #trajectoryUI{
    position: absolute;
    left:50px;
    top:50px;
    width: 400px;
    height: 420px;
    background: #545557;
    padding: 5px 5px 0;
  }
  .tra-table-body{
    width:100%;
    height:313px;
    overflow-y:scroll;
  }
  li{
    list-style-type:none;
  }

</style>
