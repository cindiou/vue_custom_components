<template>
  <div id="app">
    <swiper
      :duration="2500"
      :shouldTransition="true"
      :imgs="imgs"
      :width="width"
      :height="height"
    />
    宽&emsp;:<input
      type="number"
      name="width"
      v-model.number.lazy="width"
    /><br />
    高&emsp;:<input
      type="number"
      name="height"
      v-model.number.lazy="height"
    /><br />
    数量:<input
      type="number"
      :min="3"
      :max="9"
      name="count"
      v-model.number.lazy="count"
    />
  </div>
</template>

<script>
import Swiper from "./components/Swiper";

export default {
  name: "App",
  components: {
    Swiper,
  },
  data() {
    return {
      imgs: [],
      width: 500,
      height: 300,
      count: 3,
    };
  },
  methods: {
    getImageUrls() {
      Promise.all(
        Array.from(
          { length: this.count },
          (_, i) => `https://picsum.photos/${this.width}/${this.height}`
        ).map((url) => fetch(url))
      ).then((res) => {
        this.imgs = res.map((r) => r.url);
      });
    },
  },
  created() {
    this.$watch(
      () => [this.width, this.height, this.count],
      function (newValue, oldValue) {
        this.getImageUrls();
      },
      {
        immediate: true,
      }
    );
  },
  computed: {},
};
</script>

<style>
img {
  vertical-align: bottom;
}
* {
  margin: 0;
  padding: 0;
}
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
</style>
