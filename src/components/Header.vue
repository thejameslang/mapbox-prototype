<template>
  <header>
    <div class="container">
      <h1>Deals Brokered by CBRE in LA</h1>
    </div>
  </header>
</template>

<script>
export default {
  name: "Header",
  components: {},
  methods: {},
  data() {
    return {};
  },
  created() {},
  mounted() {},
  updated() {},
  watch: {}
};
</script>

<style lang="scss">
header {
  z-index: 2;
  top: 0;
  position: absolute;
  width: 100%;
  height: 60px;
  background-color: #000;
  color: #fff;
  border-bottom: 5px solid #00a657;
  .container {
    display: block;
    // margin: 0 auto;
    // max-width: 960px;
    margin-left: 10px;
    h1 {
      text-transform: uppercase;
      line-height: 40px;
      margin: 10px 0;
    }
  }
}
</style>