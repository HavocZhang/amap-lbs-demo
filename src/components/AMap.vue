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
  const mockData = [
    { id: 0, type: 1 },
    { id: 1, type: 1 },
    { id: 2, type: 2 },
    { id: 3, type: 2 },
    { id: 4, type: 2 },
  ];
  const typeImage = {
    1: "/1.png",
    2: "/2.png",
  };
  const typeImageClick = {
    1: "/1-c.png",
    2: "/2-c.png",
  };
  const viaMarkerList: any[] = [];
  mockData.forEach((item, index) => {
    // 以 icon URL 的形式创建一个途经点
    let viaMarker = new AMap.Marker({
      position: new AMap.LngLat(116.38 + index, 39.92 + index),
      // 将一张图片的地址设置为 icon
      icon: new AMap.Icon({
        image: (typeImage as any)[item.type],
      }),
      // 设置了 icon 以后，设置 icon 的偏移量，以 icon 的 [center bottom] 为原点
      offset: new AMap.Pixel(-13, -30),
      //用户自定义属性，支持JavaScript API任意数据类型，如Marker的id等
      extData: {
        id: item.id,
        type: item.type,
      },
    });
    viaMarker.on("click", (e: any) => {
      let id = e.target.getExtData().id;
      let type = e.target.getExtData().type;
      viaMarkerList.forEach((markerItem) => {
        let markerId = markerItem.getExtData().id;
        if (id === markerId) {
          //相同放大
          e.target.setIcon(
            new AMap.Icon({
              image: (typeImageClick as any)[type],
            })
          );
          e.target.setExtData({
            id: id,
            type: type,
            state: 1,
          });
        } else {
          //不同复原
          if (markerItem.getExtData().state === 1) {
            markerItem.setIcon(
              new AMap.Icon({
                image: (typeImage as any)[markerItem.getExtData().type],
              })
            );
          } else {
            markerItem.setIcon(
              new AMap.Icon({
                image: markerItem.getIcon()._opts.image,
              })
            );
          }
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
