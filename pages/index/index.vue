<template>
	<view>
		<view class="title">日期选择 - 示例</view>
		<button type="primary" @click="onShowDatePicker">选择日期</button>
		<mx-datepicker v-model="showPicker" format="datetime" @selected="onSelected" />
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
