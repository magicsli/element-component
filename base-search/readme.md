### `base-search`说明

**使用说明  **

```vue


<template>   
	<searchOption
      :show-q="true"
      :search-form.sync="searchForm"
      :condition="search"
      @toSearch="getRenderData"
    >
</template>

<script>
    import searchOption from '@/components/base-search'
    export default {
      components: {
        searchOption
      },
      data() {
        return {
          // 搜索相关
          searchForm: {
          },
          renderFun: 'getLiveSettle',
          search: [
            { label: '结算ID', type:'text' maxlength: 19, placeholder: '请输入编号', valueName: 'settlement_id' },
            { label: '报名订单号', type: 'text', maxlength: 19, placeholder: '请输入订单号', valueName: 'order_no' },
            { label: '活动编号', type: 'text', maxlength: 19, placeholder: '请输入编号', valueName: 'bill_no' },
            { label: '区间测试', type: 'section', maxlength: 19, valueName: 'state' },
            { label: '用户名称', type: 'text', maxlength: 15, valueName: 'user_name' },
            { label: '手机号码', type: 'number', maxlength: 11, valueName: 'phone' },
            { label: '创建时间', type: 'timepicker', valueName: 'created' },
            { label: '业务类型', type: 'select', selectList: [
              { key: '', value: '全部' },
              { key: '1', value: '直播服务' },
              { key: '2', value: '短视频服务' }
            ], valueName: 'type' }
          ]
        }
      },

      methods: {

      }
    }
    
</script>

```





#### `porps`说明



##### `searchForm` ( 绑定输入对象 )

绑定的表单输入对象, 必传属性



##### `condition` ( 条件列表 ) 

 此属性为组件**核心属性**, 用于设置其所有动态searchItem,

```javascript
/**
* @type {Array}
*/
[
  { 
     // 设置item的label属性
   	label: '结算ID', 
     
    /*
    	核心属性， 必传
    	type: text  - 普通文本输入框
    	type: number - 数字输入框( 在onInput中进行了校验, 若传入自定义处理则不进行默认数字校验)
    	type: select - 下拉选择框( 需要传入selectList( 遍历为options )  )
    	type: timepicker - 时间区间选择器 (默认) 会自动绑定三个值 valueName. valueName_start_time, valueName_end_time 
    	type: timepicker-custom   - 自定义时间区间选择器, 上列功能的优化版本
    	type: section  - 数字区间选择器, 默认绑定 valueName. valueName_min, valueName_max
    	type: default  - 预设的几个固定选择器, 限定为下拉列表
     */
   	type: 'text', 
        
    // 核心属性, 必传, 绑定在searchData中的键名,
    valueName: 'settlement_id' 
    
    // 输入字符的长度, (仅在type为number, input, 或者section中可用)
   	maxlength: 19, 
      
    // 输入框提示, 默认 ` 请输入 + label `
  	placeholder: '请输入编号',
   	
    // 下拉列表, (type: select 或 default 时生效) 传值, 以 key(值), value(显示)
    selectList: [
          { key: '', value: '全部' },
          { key: '1', value: '直播服务' },
          { key: '2', value: '短视频服务' }
	],
      
      
    // 输入框label长度 (参考element的labelwidth传入)
    labelWidth: '100px',
    
    // 预设的类型 参考组件中的预设值: payMethod, serveName
    defaultType: "payMethod"
    
  },
]

```



##### `showQ` (是否显示快捷搜索)

默认`false`,  `searchData`中传参字段: q



##### `labelWidth` (label宽度)

默认 "",  参考element中`labelWidth`的设置



##### `placeholder`( 快捷搜索提示 )

默认 "请输入关键字搜索", 只有在 `showQ` 时显示



##### `handleChange` ( change事件回调 )

change事件的回调参数, 在数据处理完时进行回调



##### `handleInput` ( input事件回调 )

input的事件回调, 在 type: number 预设了一个正则校验的默认input事件, 如果这是设置了自定义, 则以传入的input时间为主, 不再进行数字校验, 



##### `handleClear` ( clear事件回调, )

同change, 由于此操作 同时会触发`onchange`事件, 就并未对此进行额外操作, 单纯伸出的额外操作事件 







#### `listeners` 说明

##### 	`toSearch`   ( 搜索事件回调 )



