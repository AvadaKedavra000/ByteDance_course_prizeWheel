<template>
  <div
    class="modal-bg"
    v-show="show"
  >
    <div class="modal-container">
      <div class="modal-header">
        {{ title }}
      </div>
      <div class="modal-main">
        <slot></slot>
      </div>
      <div class="modal-footer">
        <button @click="submit">确认</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "modal",
  props: {
    show: {
      type: Boolean,
      default: false,
    },
    title: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      node: null,
    };
  },
  mounted() {
    this.node = document.querySelector(".modal-container");
  },
  methods: {
    submit() {
      this.$emit("submit");
    }
  },
};
</script>
<style scoped>
.modal-bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 10;
}
.modal-container {
  background: #fff;
  border-radius: 10px;
  overflow: hidden;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width:60%;
}
.modal-header {
  height: 56px;
  background: #409eff;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: move;
}
.modal-footer {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 57px;
  border-top: 1px solid #ddd;
}
.modal-footer button {
  width: 90%;
  height:55%;
  background-color:#409eff;
  color:#fff;
  border:  solid 0px;
}
.modal-main {
  padding: 15px 40px;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>