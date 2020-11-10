<template>
  <div class="handle-box">
    <div v-if="showQ" class="flex aic">
      <div class="f1">
        <el-input
          v-model="searchForm.q"
          :placeholder="placeholder"
          class="search-style mr20"
          @keyup.enter.native="handleSearch"
        >
          <el-button slot="append" icon="el-icon-search" @click="handleSearch">快速搜索</el-button>
        </el-input>
      </div>
      <div style="width: 100px;" class="tac">
        <el-link :underline="false" @click="moreOparation = !moreOparation">
          更多筛选
          <i :class="moreOparation ? 'el-icon-caret-top': 'el-icon-caret-bottom'" />
        </el-link>
      </div>
    </div>
    <transition name="fade-top">
      <div v-show="moreOparation && showQ" class="mt20">
        <el-form inline :rules="rules">
          <el-form-item
            v-for="item in condition"
            :key="item.label"
            :rules="item.rules"
            :label="item.label "
            :aria-disabled="true"
          >
            <el-input
              v-if="item.type === 'text'"
              v-model="searchData[item.valueName]"
              :disabled="item.disabled"
              type="text"
              :maxlength="item.maxlength"
              :placeholder="item.placeholder || '请输入' + item.label"
              @change="handleEventFun(item.handleChange)"
              @input="handleEventFun(item.handleInput)"
              @clear="handleEventFun(item.handleClear)"
            />
            <el-input
              v-if="item.type === 'phone'"
              v-model="searchData[item.valueName]"
              :disabled="item.disabled"
              type="phone"
              :maxlength="item.maxlength"
              :placeholder="item.placeholder || '请输入' + item.label"
              @change="handleEventFun(item.handleChange)"
              @input="handleEventFun(item.handleInput)"
              @clear="handleEventFun(item.handleClear)"
            />
            <el-select
              v-if="item.type === 'select'"
              v-model="searchData[item.valueName]"
              :disabled="item.disabled"
              :placeholder="item.placeholder || '请选择'"
              clearable
              @change="val => handleSelectChange(val, item.handleChange)"
            >
              <el-option
                v-for="selectitem in item.selectList"
                :key="selectitem.key"
                :value="selectitem.key"
                :label="selectitem.value"
              />
            </el-select>
            <el-select
              v-if="item.type === 'payMethod'"
              v-model="searchData[item.valueName]"
              :disabled="item.disabled"
              :placeholder="item.placeholder || '请选择'"
              clearable
              @change="val => handleSelectChange(val, item.handleChange)"
            >
              <el-option
                v-for="selectitem in defaultSelectList.payMethod"
                :key="selectitem.key"
                :value="selectitem.key"
                :label="selectitem.value"
              />
            </el-select>
            <el-date-picker
              v-if="item.type === 'timepicker-default' || item.type === 'timepicker'"
              v-model="searchData[item.valueName]"
              type="daterange"
              value-format="yyyy-MM-dd"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :picker-options="pickerOptions"
              align="right"
              @change="val => handleChangeTime(item.valueName, val, item.handleChange)"
            />
            <el-date-picker
              v-if="item.type === 'timepicker-custom'"
              v-model="auditedTempTime[item.valueName]"
              type="daterange"
              value-format="yyyy-MM-dd"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :picker-options="pickerOptions"
              align="right"
              @change="val => handleCustomChangeTime(item.valueName, val, item.pickerType, item.handleChange)"
            />
          </el-form-item>
          <slot />
        </el-form>
      </div>
    </transition>
  </div>
</template>
<script>
/**
 * 2020/10/20 - searchOption功能重构 -- 小利
 */
import pickerOptions from '@/mixins/pickerOptions'
const defaultSelectList = {
  payMethod: [
    { key: 1, value: '支付宝' },
    { key: 2, value: '微付支付' },
    { key: 3, value: '余额支付' },
    { key: 10, value: '银行转账' }
  ],
  serveName: [
    { key: 1, value: '全部' },
    { key: 2, value: '商家vip' },
    { key: 3, value: '团长vip' },
    { key: 10, value: '达人vip' },
    { key: 10, value: '推广升级' }
  ]
}

export default {
  mixins: [pickerOptions],
  props: {
    // 是否显示快捷搜索栏
    showQ: {
      type: Boolean,
      default: false
    },
    // 筛选条件
    condition: {
      type: Array,
      default: () => []
    },
    // 顶部搜索栏提示
    placeholder: {
      type: String,
      default: '请输入关键字搜索'
    },
    // 全局搜索规则（-- 参考element表单验证规则）
    rules: {
      type: Object,
      default: () => ({})
    },
    // 搜索表单数据
    searchForm: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      moreOparation: false, // 是否显示细节搜索
      auditedTempTime: {}, // 用于保存时间段的数组（原始类型数据）
      defaultSelectList: defaultSelectList // 默认搜索栏中的的常用下拉列表数据（ 引用规则：未设置下拉列表数据）
    }
  },
  computed: {
    searchData: {
      get() {
        return this.searchForm
      },
      set() {
        this.$emit('updata:searchForm', this.searchForm)
      }
    }
  },
  methods: {
    // 空函数， 减少内存消耗
    emptyFunction() {},
    /**
     * 事件处理，设置函数式的事件处理函数
     * @param {function} eventFun 事件处理函数
     */
    handleEventFun(eventFun) {
      // 修改日志：使用返回函数来使eventFun正常接收onChange所携带的参数； - 2020/10/21
      return typeof eventFun === 'function' ? eventFun : this.emptyFunction
    },
    /**
     * 自定义切换时间段组件
     * @param {string} valueName 字段名
     * @param {array} val 时间段切换所得的数组/null（ 0：开始时间， 1：结束时间 ）
     * @param {string} type 类型字符串，例：start-end( start代表开始时间， end代表结束时间 )
     * @param {function} callback change的回调函数
     */
    handleCustomChangeTime(valueName, val, type = 'start~end', callback) {
      console.log(valueName, val, type)
      if (val) {
        // 判断设置开始时间/结束时间位置
        if (/start/g.test(type) || /end/g.test(type)) {
          this.searchForm[valueName] = type.replace(/start/g, val[0])
            .replace(/end/g, val[1])
        }
      } else {
        this.searchForm[valueName] = ''
      }
      // 不适用上列handleEventFun判断函数， 减少函数调用（内存消耗）
      typeof callback === 'function' && callback(val)
    },
    /**
     * 默认切换时间段组件
     * @param {string} valueName 字段名
     * @param {array} val 时间段切换所得的数组/null（ 0：开始时间， 1：结束时间 ）
     * @param {function} callback change的回调函数
     */
    handleChangeTime(valueName, val, callback) {
      if (val) {
        this.searchForm[valueName + '_start_time'] = val[0]
        this.searchForm[valueName + '_end_time'] = val[1]
      } else {
        this.searchForm[valueName + '_start_time'] = ''
        this.searchForm[valueName + '_end_time'] = ''
      }
      typeof callback === 'function' && callback(val)
    },
    /**
     * 下拉列表发生变化
     * @param {string} val 下拉框所选中数据
     * @param {function} callback change的回调函数
     */
    handleSelectChange(val = '', callback) {
      typeof callback === 'function' && callback(val)
      this.handleSearch()
    },
    // 搜索操作
    handleSearch() {
      this.$listeners.toSearch({ ...this.searchForm, page: 1 })
      this.$listeners.getStatus &&
        this.$listeners.getStatus({ ...this.searchForm, status: 0 })
    }
  }
}
</script>
