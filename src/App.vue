<template>
  <div id="main-map" class="map-box">
    <div class="tool">
      <button @click="activate('Polyline')">绘制线</button>
      <button @click="deactivateDraw()">取消绘制</button>
      <button @click="deactivateEdit()">取消编辑状态</button>
      <button @click="sp()">序列化和反序列化</button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import 'ol/ol.css';
import 'ol-plot/dist/ol-plot.css';
import { Map, View } from 'ol';
import { Tile } from 'ol/layer';
import { OSM } from 'ol/source';
import { fromLonLat } from 'ol/proj';
import Plot from 'ol-plot';
import { onMounted, shallowRef, ref } from 'vue';

const map = shallowRef();
const view = shallowRef();
const plot = shallowRef();

const inited = ref(false);

const initMap = () => {
  view.value = new View({
    center: fromLonLat([113.534501, 34.441045]),
    zoom: 4,
  });
  map.value = new Map({
    target: 'main-map',
    layers: [
      new Tile({
        visible: true,
        source: new OSM({
          url: 'https://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineCommunity/MapServer/tile/{z}/{y}/{x}',
        }),
      }),
    ],
    view: view.value,
  });
  inited.value = true;

  /* eslint-disable-next-line */
  plot.value = new Plot(map.value, {
    zoomToExtent: true,
  });

  map.value.on('click', function (event) {
    // eslint-disable-next-line @typescript-eslint/no-shadow
    const feature = map.value.forEachFeatureAtPixel(
      event.pixel,
      function (feature) {
        return feature;
      }
    );
    if (feature && feature.get('isPlot') && !plot.value.plotDraw.isDrawing()) {
      plot.value.plotEdit.activate(feature);
    } else {
      plot.value.plotEdit.deactivate();
    }
  });

  // 绘制结束后，添加到FeatureOverlay显示。
  function onDrawEnd(event) {
    console.log(event);
    const feature = event.feature;
    // 开始编辑
    plot.value.plotEdit.activate(feature);
  }

  plot.value.plotDraw.on('drawStart', (e) => {
    console.log(e);
  });
  plot.value.plotDraw.on('drawEnd', onDrawEnd);

  plot.value.plotEdit.on('activePlotChange', (e) => {
    console.log(e);
  });
  plot.value.plotEdit.on('deactivatePlot', (e) => {
    console.log(e);
  });

  plot.value.on('activeTextArea', function (event) {
    const style = event.overlay.getStyle();
    console.log(style);
  });
  plot.value.on('deactivateTextArea', function (event) {
    const style = event.overlay.getStyle();
    console.log(style);
  });
};

// 指定标绘类型，开始绘制。
function activate(type) {
  if (!plot.value) {
    return;
  }
  plot.value.plotEdit.deactivate();
  plot.value.plotDraw.activate(type);
}

function deactivateDraw() {
  if (!plot.value) {
    return;
  }
  plot.value.plotDraw.deactivate();
}

function deactivateEdit() {
  if (!plot.value) {
    return;
  }
  plot.value.plotEdit.deactivate();
}

function sp() {
  if (!plot.value) {
    return;
  }
  const features = plot.value.plotUtils.getFeatures();
  console.log(JSON.stringify(features));
  plot.value.plotUtils.removeAllFeatures();
  plot.value.plotEdit.deactivate();
  plot.value.plotUtils.addFeatures(features);
}

onMounted(() => {
  initMap();
});
</script>

<style lang="less">
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
ul,
li {
  list-style: none;
}
.clearfix {
  *zoom: 1;
}
.clearfix:after {
  visibility: hidden;
  display: block;
  font-size: 0;
  content: ' ';
  clear: both;
  height: 0;
}
:focus {
  outline: none;
}
a {
  text-decoration: none;
  outline: none;
}
.map-box {
  position: absolute;
  top: 0px;
  bottom: 0px;
  left: 0px;
  right: 0px;
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  font-family: Helvetica Neue For Number, -apple-system, BlinkMacSystemFont,
    Segoe UI, Roboto, PingFang SC, Hiragino Sans GB, Microsoft YaHei,
    Helvetica Neue, Helvetica, Arial, sans-serif;
  font-size: 12px;
  line-height: 1.5;
  color: #3a3a3a;

  .tool {
    position: absolute;

    top: 10px;
    left: 50%;
    transform: translateX(-50%);

    z-index: 2;
  }
}
</style>
