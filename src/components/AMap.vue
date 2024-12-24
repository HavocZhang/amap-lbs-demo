<script setup lang="ts">
import { useScriptTag } from "@vueuse/core";
import { onMounted, ref, useTemplateRef } from "vue";

onMounted(() => {
  initMap();
});
const { load } = useScriptTag(
  "https://webapi.amap.com/maps?v=2.0&key=4816b3595e2a3a69dd720eb265413f8c"
);
const mapRef = useTemplateRef("mapRef");
const map = ref();
async function initMap() {
  await load();
  if (!mapRef.value) return;
  map.value = new AMap.Map(mapRef.value, {
    zoom: 13,
    center: [116.4, 39.92],
    resizeEnable: true,
  });
  const mockData = [0, 1, 2, 3, 4, 5, 6];
  const viaMarkerList: any[] = [];
  mockData.forEach((item, index) => {
    // 以 icon URL 的形式创建一个途经点
    let viaMarker = new AMap.Marker({
      position: new AMap.LngLat(116.38 + index, 39.92 + index),
      // 将一张图片的地址设置为 icon
      icon: new AMap.Icon({
        // 图标尺寸
        size: new AMap.Size(25, 34),
        // 图标的取图地址
        image:
          "//a.amap.com/jsapi_demos/static/demo-center/icons/dir-marker.png",
        // 图标所用图片大小
        imageSize: new AMap.Size(135, 40),
        // 图标取图偏移量
        imageOffset: new AMap.Pixel(-9, -3),
      }),
      // 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
      offset: new AMap.Pixel(-13, -30),
      //用户自定义属性，支持JavaScript API任意数据类型，如Marker的id等
      extData: {
        id: item,
      },
    });
    viaMarker.on("click", (e: any) => {
      let id = e.target.getExtData().id;
      viaMarkerList.forEach((markerItem) => {
        let markerId = markerItem.getExtData().id;
        if (id === markerId) {
          //相同放大
          e.target.setIcon(
            new AMap.Icon({
              size: new AMap.Size(25, 34),
              image:
                "//a.amap.com/jsapi_demos/static/demo-center/icons/dir-marker.png",
              imageSize: new AMap.Size(135, 40),
              imageOffset: new AMap.Pixel(-95, -3),
            })
          );
        } else {
          //不同复原
          markerItem.setIcon(
            new AMap.Icon({
              size: new AMap.Size(25, 34),
              image: markerItem.getIcon()._opts.image,
              imageSize: new AMap.Size(135, 40),
              imageOffset: new AMap.Pixel(-9, -3),
            })
          );
        }
      });
    });
    viaMarkerList.push(viaMarker);
  });

  map.value.add(viaMarkerList);
}
</script>
<template>
  <div ref="mapRef" class="w-full h-full"></div>
</template>
