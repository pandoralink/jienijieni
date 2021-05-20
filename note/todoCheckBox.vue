<template>
  <el-checkbox
    class="pl-note-checkbox"
    @change="start"
    :value="radioState"
  ></el-checkbox>
</template>

<script>
export default {
  name: "todo-check-box",
  props: {
    radioState: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      timeOutEvent: 0,
      taskState: false,
      isChange: false
    }
  },
  methods: {
    start(state) {
      // 开启定时器，定时器结束提交 taskState 的变化到数据库
      this.$emit('radioInput',state);
      this.taskState = state;
      if(this.timeOutEvent == 0 && this.isChange == false) {
        this.isChange = true;
        this.timeOutEvent = setTimeout(() => {
          // 这里面是定时器结束的逻辑
          console.log(this.taskState);
          this.timeOutEvent = 0;
          this.isChange = false;
          this.$emit("endPost",this.taskState);
        },60000);
      }
      else {
        return;
      }
    }
  }
}
</script>

<style>

</style>