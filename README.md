## DatePicker
## 介绍
Uni-App的一个支持多类型选择（日期、日期时间、时间）的时间选择插件

地址：[Uni-App插件地址](https://ext.dcloud.net.cn/plugin?id=112) | [Gitee](https://gitee.com/nullfeng/uniapp_date_and_time_selector)
## 属性说明
* `v-model` `[Boolean]` 是否显示控件（双向绑定）
* `init` `[String]` 初始化数值(要与类型对应)，默认为空
* `type` `[String]` 仅支持date/datetime/time三种，默认为datetime
## 事件说明
* `selected` `[function]` 时间选择完毕后的事件

```
onSelected(data) {
    console.log(data.value);//2019-1-1 12:11:00
}
```
其中data.value为所选值。

## 使用示例
```
<template>
	<view>
		<view class="title">日期选择 - 示例</view>
		<button type="primary" @click="onShowDatePicker">选择日期</button>
		<mx-datepicker v-model="showPicker" init="2018-12-24 18:00:59" type="datetime" @selected="onSelected" />
	</view>
</template>
<script>
	import MxDatepicker from "@/components/mx-datepicker/mx-datepicker.vue"
	export default {
		components: {
			MxDatepicker
		},
		data() {
			return {
				showPicker: false
			}
		},
		methods: {
			onShowDatePicker(){//显示
				this.showPicker = true;
			},
			onSelected(data) {//选择
				console.log(data.value);
			}
		}
	}
</script>
<style>
	.title{
		text-align: center;
		padding: 10px 0;
	}
</style>
```
## 未来支持

* 时间段选择
* 支持更多自定义格式

## 更新日志

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