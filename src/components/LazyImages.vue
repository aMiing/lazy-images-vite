<script setup>
import { onMounted, ref, nextTick, onUnmounted } from "vue";

let imageData = ref([]);
async function getMoreData(page = 0) {
  const response = await fetch(
    "https://picsum.photos/v2/list?limit=50&page=" + page
  );
  let newData = await response.json();
  newData = newData.map((e) => {
    return {
      ...e,
      download_url:
        e.download_url.split("/").slice(0, -2).join("/") + "/300/200",
    };
  });
  imageData.value = [...imageData.value, ...newData];
}
let observer = ref(null);
const root = ref(null);
onMounted(async () => {
  await getMoreData();
  await nextTick();
  InitObserve();
});

function InitObserve() {
  observer.value = new IntersectionObserver(handlerObserve, {
    root: root.value,
    rootMargin: "0px 0px 800px 0px", // 监视区向下拓展800px
  });
  addObserve();
}
function addObserve() {
  const list = document.querySelectorAll(".image-item-box");
  for (let i = 0; i < list.length; i++) {
    const element = list[i];
    observer.value.observe(element);
  }
}
function handlerObserve(entries) {
  entries.forEach(({ isIntersecting, target }) => {
    if (isIntersecting) {
      const targetImg = target.children[0];
      // console.log("target", target);
      targetImg.src = targetImg.dataset.src;
      // 修改过src属性之后，即可取消监视
      observer.value.unobserve(target);
    }
  });
}
// 关闭观察器
onUnmounted(() => {
  observer.value.disconnect();
});
</script>

<template>
  <div class="lazy-content" ref="root">
    <div class="image-item-box" v-for="item in imageData" :key="item.id">
      <!-- 最好能给src一张默认缺省图片 -->
      <img
        class="observeImg"
        src=""
        :data-src="item.download_url"
        alt="正在加载"
        :key="item.id"
        width="400"
      />
    </div>
    <!-- 拓展一下: 监视该区域出现在交叉区，向服务器请求更多数据 -->
    <div class="bottom-text">
      <!-- <span v-if="page < 50">正在为您加载更多...</span> -->
      <span>加载完毕~</span>
    </div>
  </div>
</template>

<style scoped>
.lazy-content {
  width: 100%;
  height: 100%;
  overflow-y: auto;
  /* display: flex;
  flex-direction: column;
  align-items: center; */
}
.image-item-box {
  width: 400px;
  height: 267px;
  background: #ccc;
  margin: 0 auto 1px;
}
</style>
