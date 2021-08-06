<template>
  <div>

    <wheel class="new"></wheel>
  </div>
</template>

<script>
import wheel from "./components/wheel.vue";
export default {
  name: "App",
  components: {
    wheel,
  },
  data() {
    return {

    };
  },
  methods: {
   
  },
  mounted() {

  },
};
</script>

<style>
#background {
  background: url("./assets/turntable-bg.png") no-repeat center center;
  background-size: cover;

  width: 100vw;
  height: 100vw;
}
.new {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translateX(-50%) translateY(-50%);
}
</style>
