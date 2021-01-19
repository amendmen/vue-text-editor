<template>
  <div class="select">
    <label for=""><slot /></label>
    <select :value="selected" @change="onSelectChange($event)">
      <option
        :value="option"
        v-for="(option, index) in options"
        :key="index"
        :style="[
          type != 'fontSize' ? { background: option } : {},
          option == 'black' ? { color: 'white' } : {},
        ]"
      >
        {{ option }}
      </option>
    </select>
  </div>
</template>

<script>
export default {
  props: {
    type: {
      type: String,
      required: true,
    },
    options: {
      type: Array,
      required: true,
    },
    selected: {
      type: String,
      required: false,
    },
  },
  methods: {
    onSelectChange(e) {
      this.$emit("valueChanged", { type: this.type, value: e.target.value });
    },
  },
};
</script>

<style lang="scss">
.select {
  display: flex;
  flex-flow: column;
  margin: 10px;
  select {
    height: 30px;
  }
}
</style>
