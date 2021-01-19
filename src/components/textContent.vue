<template>
  <div class="text-content">
    <span
      v-for="(item, index) in data"
      :key="index"
      :style="getStyles(item)"
      @mousedown="$emit('mouseDown', index)"
      @mouseup="$emit('mouseUp', index)"
      >{{ item.text }}</span
    >
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      required: true,
    },
  },
  methods: {
    getStyles(item) {
      let css = {
        color: item.color,
        fontSize: item.fontSize,
        backgroundColor: item.bcgColor,
      };

      return css;
    },
  },
};
</script>

<style lang="scss">
.text-content {
  padding-top: 50px;
  padding-bottom: 50px;
  border: 1px dotted #000;
  span {
    word-break: break-all;
    position: relative;
    text-shadow: 1px 1px 2px black;
  }
}
</style>
