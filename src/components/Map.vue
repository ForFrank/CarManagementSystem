<template>
  <div id="cesiumContainer">
    <div id="animationContainer" class="cesium-viewer-animationContainer" style="z-index: 100;  bottom:20px; left: 20px; float:left;"></div>
    <div id="timelineContainer" class="cesium-viewer-timelineContainer" style="z-index: 100;  right: 50px; bottom:20px; left: 188px; "></div>
    <div id="latlng_show">
        <div class="latlog" >
          <p>经度：<span id="longitude_show"></span></p>
        </div>
        <div class="latlog">
          <p>纬度：<span id="latitude_show"></span></p>
        </div>
      </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import $ from 'jquery'
  import bus from '../components/bus.vue'
  export default {
        name: "Map",
    data:function(){
      return{
        msg:''
      }
    },
      mounted(){
        var flags = {
          realtimeFlag:true
      };
        var _this = this;
        var MapUrl=window.g.MAP_URL;
        //创建viewer实例
        var viewer = new Cesium.Viewer('cesiumContainer', {
          animation: false, //是否创建动画小器件，左下角仪表
          baseLayerPicker: false, //是否显示图层选择器
          fullscreenButton: true, //是否显示全屏按钮
          geocoder: false, //是否显示geocoder小器件，右上角查询按钮
          homeButton: false, //是否显示Home按钮
          infoBox: true, //是否显示信息框
          sceneModePicker: false, //是否显示3D/2D选择器
          selectionIndicator: true, //是否显示选取指示器组件
          timeline: false, //是否显示时间轴
          navigationHelpButton: false, //是否显示右上角的帮助按钮
          scene3DOnly: false, //如果设置为true，则所有几何图形以3D模式绘制以节约GPU资源
          clock: new Cesium.Clock(), //用于控制当前时间的时钟对象
          selectedImageryProviderViewModel: undefined, //当前图像图层的显示模型，仅baseLayerPicker设为true有意义
          imageryProviderViewModels: Cesium.createDefaultImageryProviderViewModels(), //可供BaseLayerPicker选择的图像图层ProviderViewModel数组
          selectedTerrainProviderViewModel: undefined, //当前地形图层的显示模型，仅baseLayerPicker设为true有意义
          terrainProviderViewModels: Cesium.createDefaultTerrainProviderViewModels(), //可供BaseLayerPicker选择的地形图层ProviderViewModel数组
          vrButton: false,
          shadows: true,
          fullscreenElement: document.getElementById("cesiumContainer"), //全屏时渲染的HTML元素,
          useDefaultRenderLoop: true, //如果需要控制渲染循环，则设为true
          targetFrameRate: undefined, //使用默认render loop时的帧率
          showRenderLoopErrors: true, //如果设为true，将在一个HTML面板中显示错误信息
          automaticallyTrackDataSourceClocks: true, //自动追踪最近添加的数据源的时钟设置
          contextOptions: undefined, //传递给Scene对象的上下文参数（scene.options）
          sceneMode: Cesium.SceneMode.SCENE2D, //初始场景模式
          mapProjection: new Cesium.WebMercatorProjection(), //地图投影体系
          dataSources: new Cesium.DataSourceCollection(),//需要进行可视化的数据源的集合
          imageryProvider: new Cesium.UrlTemplateImageryProvider({
            credit: 'Topsci Mapscloud.',
            url: MapUrl
          }),
        });


        viewer._cesiumWidget._creditContainer.style.display = "none";//去除cesium logo


        //地图初始化位置
        viewer.camera.flyTo({
          destination: Cesium.Cartesian3.fromDegrees(116.38, 39.90, 150000)
        });
        //实时显示经纬度
        var longitude_show=document.getElementById('longitude_show');
        var latitude_show=document.getElementById('latitude_show');
        var canvas=viewer.scene.canvas;
        var ellipsoid=viewer.scene.globe.ellipsoid;
        var handler = new Cesium.ScreenSpaceEventHandler(canvas);
        handler.setInputAction(function(movement){
          var cartesian=viewer.camera.pickEllipsoid(movement.endPosition, ellipsoid);
          if(cartesian){
            var cartographic=viewer.scene.globe.ellipsoid.cartesianToCartographic(cartesian);
            var lat_String=Cesium.Math.toDegrees(cartographic.latitude).toFixed(4);
            var log_String=Cesium.Math.toDegrees(cartographic.longitude).toFixed(4);
            longitude_show.innerHTML=log_String;
            latitude_show.innerHTML=lat_String;
          }
        },Cesium.ScreenSpaceEventType.MOUSE_MOVE);

        //地图缩放控件
        /*var options = {};
        options.defaultResetView = Cesium.Rectangle.fromDegrees(71, 3, 90, 14);
        // Only the compass will show on the map
        options.enableCompass = true;
        options.enableZoomControls = false;
        options.enableDistanceLegend = false;
        options.enableCompassOuterRing = true;*/
        var test= new Cesium.Cartographic.fromDegrees(116.38, 39.90, 150000);
        viewer.extend(Cesium.viewerCesiumNavigationMixin,
          {defaultResetView:test,
           enableDistanceLegend:true
          });

        //车辆位置实时监控
        var queryCarPositionUrl=window.g.queryCarPosition_URL;
        var datajson = [];
        var times;
        var point1;
        var point2;
        function querytest(){
          flags.realtimeFlag = false;
          viewer.entities.removeAll();
          viewer.dataSources.removeAll();
          $("div.cesium-viewer-animationContainer").hide(500);
          $(".cesium-viewer-timelineContainer").hide(500);
          _this.flag=true
          datajson=[];
          var pt1 = new Cesium.Cartesian2(0,0);
          var pt2= new Cesium.Cartesian2(viewer.canvas.clientWidth, viewer.canvas.clientHeight);
          var pick1= viewer.scene.globe.pick(viewer.camera.getPickRay(pt1), viewer.scene);
          var pick2= viewer.scene.globe.pick(viewer.camera.getPickRay(pt2), viewer.scene);
          var geoPt1= viewer.scene.globe.ellipsoid.cartesianToCartographic(pick1);
          var geoPt2= viewer.scene.globe.ellipsoid.cartesianToCartographic(pick2);
          var point1=[geoPt1.longitude / Math.PI * 180,geoPt1.latitude / Math.PI * 180];
          var point2=[geoPt2.longitude / Math.PI * 180,geoPt2.latitude / Math.PI * 180];
          axios.get(queryCarPositionUrl, {params: {left: 0,right:200,bottom:0,top:200}}).
          // axios.get(queryCarPositionUrl, {params: {left: point1[0],right:point2[0],bottom:point2[1],top:point1[1]}}).
          then(function(response){
            datajson = response['data'].data;
            viewer.camera.flyTo({
              destination: Cesium.Cartesian3.fromDegrees(parseFloat(datajson[0].longitude),parseFloat(datajson[0].latitude), 100000),});
            var obj = datajson.length
            for (var i = 0; i <obj; i++){
              createRoutePoint(datajson[i]);
            };
            function createRoutePoint(item) {
              var description = '<table class="cesium-infoBox-defaultTable cesium-infoBox-defaultTable-lighter"><tbody>';
              description += '<tr><th>' + "车速" + '</th><td>' + item.mobileyeChinaVelocity + '</td></tr>';
              description += '<tr><th>' + "油量" + '</th><td>' + item.oilMass + '</td></tr>';
              description += '<tr><th>' + "经度" + '</th><td>' + item.longitude + '</td></tr>';
              description += '<tr><th>' + "维度" + '</th><td>' + item.latitude + '</td></tr>';
              description += '</tbody></table>';
              //显示全部点
              viewer.entities.add({
                name: '车辆ID：'+item.id,
                description:description,
                position: Cesium.Cartesian3.fromDegrees(parseFloat(item.longitude),parseFloat(item.latitude), 5000),
                label: {
                  text: item.deviceCode,
                  font: '12px Microsoft YaHei',
                  style: Cesium.LabelStyle.FILL_AND_OUTLINE,
                  fillColor: Cesium.Color.WHITE,
                  outlineColor: Cesium.Color.BLACK,
                  outlineWidth: 4,
                  verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                  pixelOffset: new Cesium.Cartesian2(0, -9),
                  translucencyByDistance: new Cesium.NearFarScalar(3000, 1.0, 4000000, 0.0),
                  height: 13000
                },
                billboard: {
                  image: './static/car.png',//汽车标识
                  width: 50,
                  height: 30,
                  translucencyByDistance: new Cesium.NearFarScalar(2000000, 1.0, 4000000, 0.0)
                },
              });
            }
            // $("body").undelegate();
          }).catch(function(error){
            console.log(error);
          });


          //定时轮询
          clearInterval(times);
          times = setInterval(function () {
            //获取当前视图范围
            var pt1 = new Cesium.Cartesian2(0,0);
            var pt2= new Cesium.Cartesian2(viewer.canvas.clientWidth, viewer.canvas.clientHeight);
            var pick1= viewer.scene.globe.pick(viewer.camera.getPickRay(pt1), viewer.scene);
            var pick2= viewer.scene.globe.pick(viewer.camera.getPickRay(pt2), viewer.scene);
            var geoPt1= viewer.scene.globe.ellipsoid.cartesianToCartographic(pick1);
            var geoPt2= viewer.scene.globe.ellipsoid.cartesianToCartographic(pick2);
            point1=[geoPt1.longitude / Math.PI * 180,geoPt1.latitude / Math.PI * 180];
            point2=[geoPt2.longitude / Math.PI * 180,geoPt2.latitude / Math.PI * 180];
            //调用实时位置查询接口
            axios.get(queryCarPositionUrl, {params: {left: point1[0],right:point2[0],bottom:point2[1],top:point1[1]}}).
            then(function(response){
              datajson = response['data'].data;
              // console.log(datajson);
              // $("body").undelegate();
            }).catch(function(error){
              console.log(error);
            });
            viewer.entities.removeAll();
            viewer.dataSources.removeAll();
            function createRoutePoint(item) {
              var description = '<table class="cesium-infoBox-defaultTable cesium-infoBox-defaultTable-lighter"><tbody>';
              description += '<tr><th>' + "车速" + '</th><td>' + item.mobileyeChinaVelocity + '</td></tr>';
              description += '<tr><th>' + "油量" + '</th><td>' + item.oilMass + '</td></tr>';
              description += '<tr><th>' + "经度" + '</th><td>' + item.longitude + '</td></tr>';
              description += '<tr><th>' + "维度" + '</th><td>' + item.latitude + '</td></tr>';
              description += '</tbody></table>';
              //实时显示点
              viewer.entities.add({
                name: '车辆ID：'+item.id,
                description:description,
                position: Cesium.Cartesian3.fromDegrees(parseFloat(item.longitude),parseFloat(item.latitude), 3000),
                label: {
                  text: item.deviceCode,
                  font: '12px Microsoft YaHei',
                  style: Cesium.LabelStyle.FILL_AND_OUTLINE,
                  fillColor: Cesium.Color.WHITE,
                  outlineColor: Cesium.Color.BLACK,
                  outlineWidth: 4,
                  verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                  pixelOffset: new Cesium.Cartesian2(0, -9),
                  translucencyByDistance: new Cesium.NearFarScalar(3000, 1.0, 4000000, 0.0),
                  height: 13000
                },
                billboard: {
                  image: './static/car.png',//汽车标识
                  width: 50,
                  height: 30,
                  translucencyByDistance: new Cesium.NearFarScalar(7000000, 1.0, 9000000, 0.0)
                },
              });
              // console.log(parseFloat(datajson[0].longitude),parseFloat(datajson[0].latitude),111111111111111111);
            }
            var obj = datajson.length
            for (var i = 0; i <obj; i++){
              createRoutePoint(datajson[i]);
            };
            datajson=[];

            if (flags.realtimeFlag === true) {
              flags.realtimeFlag=false;
              datajson=[];

              viewer.entities.removeAll();
              viewer.dataSources.removeAll();
              clearInterval(times);
            };
          },3000);



        }

        $("body").delegate('#realtime', 'click', querytest);

        $("body").delegate('#history', 'click', function () {
          viewer.entities.removeAll();
          viewer.dataSources.removeAll();
          flags.realtimeFlag=true;
        });

        $("body").delegate('#buttonclick', 'click', function () {
          viewer.entities.removeAll();
          viewer.dataSources.removeAll();
          $(".cesium-viewer-animationContainer").hide(500);
          $(".cesium-viewer-timelineContainer").hide(500);

        });
        $("body").delegate('#trajectorybutton', 'click', function () {
          viewer.entities.removeAll();
          viewer.dataSources.removeAll();
          $(".cesium-viewer-animationContainer").hide(500);
          $(".cesium-viewer-timelineContainer").hide(500);

        });
        $("body").delegate('#trajectoryelbutton', 'click', function () {
          viewer.entities.removeAll();
          viewer.dataSources.removeAll();
          $(".cesium-viewer-animationContainer").hide(500);
          $(".cesium-viewer-timelineContainer").hide(500);

        });

        //显示历史轨迹
        var tracks = [];
        bus.$on('trajectory',function(msg){
          _this.msg=msg;
          tracks = [];
          $.each(_this.msg,function(idx,obj){
            tracks.push(idx,parseFloat(obj.longitude),parseFloat(obj.latitude),0);
          });
          $("div.cesium-viewer-animationContainer").hide(500);
          $(".cesium-viewer-timelineContainer").hide(500);

          viewer.entities.removeAll();
          viewer.dataSources.removeAll();

          //时间格式转换
          var trackTimeUtcs =(new Date(msg[0].trackTime)).toISOString();
          var trackTimeUtce =(new Date(msg[msg.length-1].trackTime)).toISOString();
          console.log( trackTimeUtcs);
          console.log( trackTimeUtce);

          /*var start = Cesium.JulianDate.fromIso8601('2015-07-30');
          var end = Cesium.JulianDate.fromIso8601('2017-06-17');
          // viewer.timeline.zoomTo(start, end);
          var clock = viewer.clock;
          clock.startTime = start;
          clock.endTime = end;
          clock.currentTime = start;
          clock.clockRange = Cesium.ClockRange.LOOP_STOP;*/
          var mapDestinationCenter = Cesium.Cartesian3.fromDegrees(parseFloat(msg[1].longitude),
            parseFloat(msg[1].latitude), 2000);
          viewer.camera.flyTo({
            destination: mapDestinationCenter,});
          var modelInfo = "<!--HTML--><table class='cesium-infoBox-defaultTable'>";
          modelInfo += "<tr><td>设备编码:</td><td>" + msg[0].deviceCode + "</td></tr>";
          modelInfo += "<tr><td>开始时间:</td><td>" + msg[0].trackTime + "</td></tr>";
          modelInfo += "<tr><td>结束时间:</td><td>" + msg[msg.length-1].trackTime + "</td></tr>";
          modelInfo += "</table>";
          var czml = [{
            "id" : "document",
            "name" : "CZML Path",
            "version" : "1.0",
            "clock": {
              "interval": trackTimeUtcs + "/" + trackTimeUtce,
              // "currentTime": trackTimeUtcs,
              "multiplier": 1
            }
          }, {
            "id" :"point",
            "name" :"设备：" + msg[0].deviceCode,
            "description" : modelInfo,
            // "availability" : "2012-08-04T10:00:00Z/2012-08-04T15:00:00Z",
            "availability" : trackTimeUtcs + "/" + trackTimeUtce,
            "path" : {
              "material" : {
                "polylineOutline" : {
                  "color" : {
                    "rgba" : [255, 0, 255, 255]
                  },
                  "outlineColor" : {
                    "rgba" : [0, 255, 255, 255]
                  },
                  "outlineWidth" : 4
                }
              },
              "width" : 5,
              "leadTime" : 0,
              "trailTime" : 1000,
              "resolution" : 0
            },
            "billboard" : {
              "image" :'./static/car.png',
              "width": 50,
              "height": 30,
            },
            "position" : {
              // "epoch" : "2012-08-04T10:00:00Z",
              "epoch" : trackTimeUtcs,
              "cartographicDegrees" : tracks
            }
          }];
          function onTimelineScrubfunction(e) {
            var clock = e.clock;
            clock.currentTime = e.timeJulian;
            clock.shouldAnimate = true;
          }
          var clockViewModel = new Cesium.ClockViewModel(viewer.clock);
          var viewModel = new Cesium.AnimationViewModel(clockViewModel);
          setTimeout(function() {
            var wigth = new Cesium.Animation('animationContainer',viewModel);
          }, 2000);
          $(".cesium-viewer-animationContainer").show(500);

          // var viewModelT = new Cesium.Timeline(clockViewModel);
          setTimeout(function() {
            var timeline = new Cesium.Timeline('timelineContainer',viewer.clock);
            timeline.addEventListener('settime', onTimelineScrubfunction, false);
            timeline.zoomTo(viewer.clock.startTime, viewer.clock.stopTime);
          }, 2000);
          $(".cesium-viewer-timelineContainer").show(500);
          viewer.dataSources.add(Cesium.CzmlDataSource.load(czml));
          // viewer.clock.multiplier = 1;
          viewer.clock.shouldAnimate = true;
          setTimeout(function() {
            viewer.trackedEntity = null;
          }, 1000);
        });
      },
    }
</script>

<style>
  #cesiumContainer {
    height: 100%;
    overflow: hidden;
    /*overflow: hidden;*/
  }
  #latlng_show {
    width:300px;
    height:30px;
    position:absolute;
    bottom:45px;
    right:160px;
    z-index:1;
    font-size:15px;
    float: right;
    overflow: hidden;
  }
  .latlog {
    width:140px;
    height:30px;
    float:right;
  }
  p{
    color: white;
  }
</style>
