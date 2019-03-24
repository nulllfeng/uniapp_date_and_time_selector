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
		<date-picker :show="showPicker" :type="type" :value="value" :show-seconds="true" @confirm="onSelected" @cancel="onSelected" />
	</view>
</template>
<script>
	import DatePicker from "@/components/datepicker/datepicker.vue";
	export default {
		components: {
			DatePicker
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
				this.showPicker = false;
				if(e) {
					this[this.type] = e.value; 
					console.log(e);
				}
			}
		}
	}
</script>
<style>
	.test{
		text-align: center;
		padding: 10px 0;
	}
	button{
		margin: 20upx;
		font-size: 28upx;
	}
</style>
