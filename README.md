## DatePicker
## 介绍
全新2.0的版本。支持日期、时间、日期时间、日期范围和日期时间范围选择。   
地址：[Uni-App插件地址](https://ext.dcloud.net.cn/plugin?id=112) | [Gitee](https://gitee.com/nullfeng/uniapp_date_and_time_selector)
## 属性说明
|属性|类型|默认值|说明|
|--	|--	|--	|-- |
|show|Boolean|false|是否显示|
|type|String|date|类型，可选值：date（日期）、time（时间）、datetime（日期时间）、range（日期范围）、rangetime（日期时间范围）|
|format|String||自定义格式|
|value|String, Array|当前时间|默认显示的值|
|showSeconds|Boolean|false|是否显示秒（针对type为datetime或time时生效）|
|color|String|#409eff|选择控件的颜色|
|showHoliday|Boolean|true|是否显示公历节日|
|@confirm|event||确认选择事件，接受的参数={value:'所选值',date:'原来的日期对象'}|
|@cancel|event||取消选择事件|
### value说明
这个参数传入的值必须是type对应格式
如果`type="time"`，则需要需要传`12:00`这种格式   
如果是日期的话，则需要传入能解析的日期格式
### format格式说明
|格式|含义|
|--	|--	|
|y|年|
|m|月|
|d|日|
|h|时|
|i|分|
|s|秒|
示例：yyyy/mm/dd hh:ii:ss => 2019/03/26 23:39
## 使用示例
```
<template>
	<view>
		<view class="test">
			<view>日期选择 - 示例</view>
			{{date}}
			<button type="primary" @click="onShowDatePicker('date')">选择日期</button>
			{{time}}
			<button type="primary" @click="onShowDatePicker('time')">选择时间</button>
			{{datetime}}
			<button type="primary" @click="onShowDatePicker('datetime')">选择日期时间</button>
			{{range}}
			<button type="primary" @click="onShowDatePicker('range')">选择日期范围</button>
			{{rangetime}}
			<button type="primary" @click="onShowDatePicker('rangetime')">选择日期时间范围</button>
		</view>
		<mx-date-picker :show="showPicker" :type="type" :value="value" :show-seconds="true" @confirm="onSelected" @cancel="onSelected" />
	</view>
</template>
<script>
	import MxDatePicker from "@/components/mx-datepicker/mx-datepicker.vue";
	export default {
		components: {
			MxDatePicker
		},
		data() {
			return {
				showPicker: false,
				date: '2019/01/01',
				time: '15:00:12',
				datetime: '2019/01/01 15:00:12',
				range: ['2019/01/01','2019/01/06'],
				rangetime: ['2019/01/08 14:00','2019/01/16 13:59'],
				type: 'date',
				value: ''
			}
		},
		methods: {
			onShowDatePicker(type){//显示
				this.type = type;
				this.showPicker = true;
				this.value = this[type];
			},
			onSelected(e) {//选择
				//e.value 为选择的值
				//e.date 为原始的date对象
				this.showPicker = false;
				if(e) {// 因为cancel事件也绑定了这个函数，而且canel事件传的参数false
					this[this.type] = e.value; 
					console.log(e);
				}
			}
		}
	}
</script>
```
## 说明
目前测试的可能不是很全面，如果有BUG或者更好的建议请在评论区反馈。

## 更新日志
v2.0.0
* 全新版本   

v1.0.3
* 优化时间选择（解决时间不能选择等问题）
* 增加默认时间和显示动画
* 之前的`format`属性更名为`type`

v1.0.1   
* 紧急修复被系统样式覆盖导致的兼容性问题

v1.0.0   
* 增加时间选择
* 支持选择类型

v0.0.7
* 完成日期选择