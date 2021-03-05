<template>
  <el-input type="text" v-bind="$attrs" v-on="{...$listeners, input:handleInput }" />
</template>

<script>
export default {
  name: 'ElNumber',
  methods: {
    handleInput(event) {
      console.log(event)
      const inputValue = event.replace(/[^0-9\.]/g, '')
      this.$emit('input', inputValue)
    }
  }
}
</script>

<style lang="scss" scoped>
/*在谷歌下移除input[number]的上下箭头*/
 /deep/ input[type='number']::-webkit-outer-spin-button,
 /deep/ input[type='number']::-webkit-inner-spin-button{
    -webkit-appearance: none !important;
}
/*在firefox下移除input[number]的上下箭头*/
/deep/ input[type="number"]{
    -moz-appearance:textfield;
}
</style>
