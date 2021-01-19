<template>
  <div class="editor" v-if="selected">
    <Select
      :type="'fontSize'"
      :options="fontSizes"
      :selected="selected ? selected.fontSize : null"
      @valueChanged="$emit('valueChanged', $event)"
      >font size</Select
    >
    <Select
      :type="'color'"
      :options="colors"
      :selected="selected ? selected.color : null"
      @valueChanged="$emit('valueChanged', $event)"
      >color</Select
    >
    <Select
      :type="'bcgColor'"
      :options="colors"
      :selected="selected ? selected.bcgColor : null"
      @valueChanged="$emit('valueChanged', $event)"
      >bacground</Select
    >
  </div>
  <div v-else class="notselected">text not selected</div>
</template>

<script>
import Select from "./select";
export default {
  props: {
    selected: {
      type: Object,
      required: false,
    },
  },
  components: {
    Select,
  },
  data() {
    return {
      colors: ["red", "gold", "white", "black", "green", "pink", "blue"],
      fontSizes: ["40px", "45px", "50px", "55px", "60px", "65px"],
    };
  },
};
</script>

<style lang="scss">
.editor,
.notselected {
  display: flex;
  justify-content: center;
  background-color: #eee;
}
.notselected {
  padding-top: 25px;
  padding-bottom: 25px;
}
</style>
