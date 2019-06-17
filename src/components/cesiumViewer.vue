<template>
  <div id="cesiumContainer"></div>
</template>

<script>
import Cesium from 'cesium/Cesium';
import 'cesium/Widgets/widgets.css';

export default {
  name: "cesiumViewer",
  data() {
    return {
      viewer:null
    };
  },
  mounted() {
    const _this = this;
    _this.viewer = new Cesium.Viewer('cesiumContainer', {
      animation : false,
      timeline : false,
      navigationHelpButton : false
    });
    // 删除默认的影像图层
    _this.viewer.imageryLayers.remove(_this.viewer.imageryLayers.get(0));
    // 增加谷歌影像底图
    _this.viewer.imageryLayers.addImageryProvider(new Cesium.UrlTemplateImageryProvider({
      url: 'http://www.google.cn/maps/vt?lyrs=s&x={x}&y={y}&z={z}',
      tilingScheme: new Cesium.WebMercatorTilingScheme()
    }));
    // 开启全球光照
    _this.viewer.scene.globe.enableLighting = true;
    // 创建相机初始位置和朝向
    let initialPosition = new Cesium.Cartesian3.fromDegrees(-73.998114468289017509, 40.674512895646692812, 2631.082799425431);
    let initialOrientation = new Cesium.HeadingPitchRoll.fromDegrees(7.1077496389876024807, -31.987223091598949054, 0.025883251314954971306);
    let homeCameraView = {
        destination : initialPosition,
        orientation : {
          heading : initialOrientation.heading,
          pitch : initialOrientation.pitch,
          roll : initialOrientation.roll
      }
    };
    // 设置视图
    _this.viewer.scene.camera.setView(homeCameraView);
    // 增加相机飞行动画参数
    homeCameraView.duration = 2.0;
    homeCameraView.maximumHeight = 2000;
    homeCameraView.pitchAdjustHeight = 2000;
    homeCameraView.endTransform = Cesium.Matrix4.IDENTITY;
    // Override the default home button
    _this.viewer.homeButton.viewModel.command.beforeExecute.addEventListener(function (e) {
        e.cancel = true;
        _this.viewer.scene.camera.flyTo(homeCameraView);
    });
    // 加载道路标线数据
    this.loadData(_this.viewer);
  },
  methods:{
    loadData: function (viewer) {
      let _this = this;
      this.$axios.get('http://localhost:8080/static/data/markLink.geojson').then(function (rep) {
        let features = rep.data.features;
        let cartesianPositions = [];
        for(let n = 0; n < features.length; n++){
          let lineCoors = features[n].geometry.coordinates;
          let position = [];
          for(let i = 0; i < lineCoors.length; i++){
            for(let j = 0; j < 3; j++){
              position.push(parseFloat(lineCoors[i][j]));
            }
          }
          cartesianPositions.push(new Cesium.Cartesian3.fromDegreesArrayHeights(position));
        }
        for(let m = 0; m < 1000; m++){
          viewer.entities.add({
            name : 'markLink',
            id: m,
            polyline : {
              positions : cartesianPositions[m],
              width : 5,
              material : new Cesium.PolylineArrowMaterialProperty(Cesium.Color.ORANGE)
            }
          })
        }
        viewer.zoomTo(viewer.entities);
      })
    }
  }
};
</script>

<style>
</style>
