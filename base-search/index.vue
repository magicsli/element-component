<template>
  <div class="handle-box">
    <div v-if="showQ" class="flex aic">
      <div class="f1">
        <el-input
          v-model.trim="searchForm.q"
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
      <div v-show="moreOparation || !showQ" class="mt20">
        <el-form inline :label-width="labelWidth">
          <el-form-item
            v-for="item in condition"
            :key="item.label"
            :label="item.label"
            :label-width="item.labelWidth"
            :aria-disabled="true"
          >
            <el-input
              v-if="item.type === 'text'"
              v-model.trim="searchData[item.valueName]"
              :disabled="item.disabled"
              type="text"
              :maxlength="item.maxlength"
              :placeholder="item.placeholder || '请输入' + item.label"
              @change="handleEventFun($event, item.handleChange)"
              @input="handleEventFun($event, item.handleInput)"
              @clear="handleEventFun($event, item.handleClear)"
            />
            <el-input
              v-if="item.type === 'number'"
              v-model.trim="searchData[item.valueName]"
              :disabled="item.disabled"
              type="text"
              :maxlength="item.maxlength"
              :placeholder="item.placeholder || '请输入' + item.label"
              @change="handleEventFun($event, item.handleChange)"
              @input="item.handleInput ? handleEventFun($event, item.handleInput) : searchData[item.valueName] = searchData[item.valueName].replace(/[^\d]/g,'')"
              @clear="handleEventFun($event, item.handleClear)"
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
              v-if="item.type === 'default'"
              v-model="searchData[item.valueName]"
              :disabled="item.disabled"
              :placeholder="item.placeholder || '请选择'"
              clearable
              @change="val => handleSelectChange(val, item.handleChange)"
            >
              <el-option
                v-for="selectitem in defaultSelectList[item.defaultType]"
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
            <div v-if="item.type === 'section'" class="section flex">
              <el-input
                v-model.trim="searchData[item.valueName + '_min']"
                :disabled="item.disabled"
                type="text"
                :maxlength="item.maxlength"
                placeholder="最小值"
                @change="handleSectionFun($event, item.valueName, item.handleChange, item.changeRule)"
                @input="item.handleInput ? handleEventFun($event, item.handleInput) :searchData[item.valueName + '_min'] =searchData[item.valueName + '_min'].replace(/[^\d]/g,'')"
                @clear="handleEventFun($event, item.handleClear)"
              />
              <span class="ml15 mr15">至</span>
              <el-input
                v-model.trim="searchData[item.valueName + '_max']"
                :disabled="item.disabled"
                type="text"
                :maxlength="item.maxlength"
                :min="searchData[item.valueName + '_min']"
                placeholder="最大值"
                @change="handleSectionFun($event, item.valueName, item.handleChange, item.changeRule)"
                @input="item.handleInput ? handleEventFun($event, item.handleInput) :searchData[item.valueName + '_max'] =searchData[item.valueName + '_max'].replace(/[^\d]/g,'')"
                @clear="handleEventFun($event, item.handleClear)"
              />
            </div>
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
/**
 * @props options:
 *    - condition 核心属性：用于设置需要添加的搜索条件
 *    - searchForm 核心属性：绑定与整个from的数据
 *    - placeholder 快捷搜索栏的提示文案
 *    - showQ 是否显示快捷搜索栏（保留前版本风格）
 *    - rules 全局form规则
 *    - labelWidth  label宽度
 */
export default {
  mixins: [pickerOptions],
  props: {
    // 是否显示快捷搜索栏
    showQ: {
      type: Boolean,
      default: false
    },
    labelWidth: {
      type: String,
      default: ''
    },
    // 筛选条件
    condition: {
      type: Array,
      default: () => [],
      required: true,
      validator(value) {
        const must = ['type', 'valueName']
        // 判断是否拥有未设置必传值的item,
        const isNoType = !value.find(item => {
          // 循环判断每一个item中是否全部包含需要的必传属性
          let res = false
          must.forEach(mustItem => {
            if (!item[mustItem]) {
              console.error('base-search组件中缺少' + must.join(', ') + '必传属性')
              res = true
            }
          })
          return res
        })
        return isNoType
      }
    },
    // 顶部搜索栏提示
    placeholder: {
      type: String,
      default: '请输入关键字搜索'
    },
    // 搜索表单数据
    searchForm: {
      type: Object,
      default: () => {},
      required: true
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

    /**
     * 事件处理，设置函数式的事件处理函数
     * @param {event} event 事件对象
     * @param {function} eventFun 事件处理函数
     */
    handleEventFun($event, eventFun) {
      /*  2020/11/23  -直接携带其参数进行判断触发；*/
      typeof eventFun === 'function' && eventFun($event)
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
      // 自定义change回调
      this.handleEventFun(val, callback)
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

      // 自定义change回调
      this.handleEventFun(val, callback)
    },
    /**
     * 下拉列表发生变化
     * @param {string} val 下拉框所选中数据
     * @param {function} callback change的回调函数
     */
    handleSelectChange(val = '', callback) {
      this.handleEventFun(val, callback)

      // 经过参考， 不需要在下拉框修改时进行search操作 - 2020/11/23
      // this.handleSearch()
    },
    /**
     * 下拉列表发生变化
     * @param {event} event input事件对象
     * @param {string} valueName 需要自定义修正的键名
     * @param {function} callback change的回调函数
     * @param {string} changeRule 最小值和最大值的连接符 “ 2_3 ” 即最小2, 最大30 默认 " _ "
     */
    handleSectionFun(event, valueName, callback, changeRule = '_') {
      /*
        if (  this.searchData[valueName + '_min'] > this.searchData[valueName + '_max']) {
          // 如果min大于max， 则提醒用户并清除max的值让用户重新输入
          this.$message.warning("最大值不能小于最小值")
          this.searchData[valueName + '_max'] = '';
          this.searchData[valueName] = this.searchData[valueName + '_min'] + changeRule + ''
          return
        }
      */
      this.searchData[valueName] = this.searchData[valueName + '_min'] + changeRule + this.searchData[valueName + '_max']

      // 自定义change回调
      this.handleEventFun(this.searchData[valueName], callback)
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
