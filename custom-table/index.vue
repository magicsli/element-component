<template>
  <div class="li-table">
    <div v-if="$slots.header" class="li-table-header">
      <slot name="header" />
    </div>
    <div v-else class="li-table-header">
      <el-checkbox v-if="showAllCheck" v-model="checkedAll" class="check-all" @change="handleAllCheck" />
      <div
        v-for="item in columns"
        :key="item.label"
        :class="{
          'li-table-title': true,
          'li-table-title-fix': item.fixed
        }"
        :style="`width: ${item.width}; min-width:${item.minWidth}; text-align:${item.align}`"
      >
        {{ item.label }}
      </div>
    </div>
    <div class="li-table-content">
      <div v-for="(item, index) in checkList" :key="index" class="li-table-content-item">
        <slot :index="index" :row="item" name="colum" />
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 自定义表格 (双行)
 * @props colum 表头列表, 例:  [ {label: "姓名"} ]. 其中label必传
 *
 */
export default {
  props: {
    columns: {
      type: Array,
      default: () => [],
      validator: (value) => !!value.find((item) => item.label)
    },
    data: {
      type: Array,
      default: () => []
    },
    showAllCheck: {
      type: Boolean,
      default: false
    },
    isAllChecked: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      checkedAll: false, // 是否全选
      checkList: [] // 选中列表, 监听data进行所有的操作
    }
  },
  watch: {
    checkList: {
      deep: true,
      handler: function() {
        // 如果没有设定选中框, 则不存在修改内部的data, 直接弹出
        // if (!this.ischeck) return
        const checkedList = this.checkList.filter(item => item.checked)
        this.$emit('selection-change', checkedList)
        this.checkedAll = checkedList.length === this.checkList.length
      }
    }
  },
  created() {
    console.log(this)
    this.checkedAll = this.isAllChecked
    this.checkList = this.$props.data.map(item => {
      item.checked = this.isAllChecked
      return item
    })
  },
  methods: {
    handleAllCheck(value) {
      this.checkList.forEach(item => {
        item.checked = this.checkedAll
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.li-table {
  padding: 20px 0;
  width: 100%;
  .li-table-header {
    display: flex;
    justify-content: space-around;
    width: 100%;
    padding: 10px 0;
    border-bottom: 1px solid #ebeef5;
    .check-all {
      width: 60px;
      padding-left: 28px;
      box-sizing: border-box;
    }
    .li-table-title {
      flex: 1;
    }
  }
}
</style>
