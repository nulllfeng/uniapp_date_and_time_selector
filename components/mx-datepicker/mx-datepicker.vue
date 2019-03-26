<template>
	<view v-if="isShow" class="mx-date-picker">
		<view v-if="type!='time'" class="mx-date-picker-modal">
			<view>
				<text class="mx-date-picker-press mx-date-picker-icon mx-date-picker-icon-zuozuo" @click="onSetYear(-1)"></text>
				<text class="mx-date-picker-press mx-date-picker-icon mx-date-picker-icon-zuo" @click="onSetMonth(-1)"></text>
				<text>{{Year}}年{{Month}}月</text>
				<text class="mx-date-picker-press mx-date-picker-icon mx-date-picker-icon-you" @click="onSetMonth(+1)"></text>
				<text class="mx-date-picker-press mx-date-picker-icon mx-date-picker-icon-youyou" @click="onSetYear(+1)"></text>
			</view>
			<view>
				<view v-for="(week,index) in weeks" :key="index - 7"><view data-pointer="true"><text>{{week}}</text></view></view>
				<view v-for="(day,index) in days" :key="index" @click="onCheckedDay(day, index)" class="mx-date-picker-press">
					<view data-bg="true" :data-range="day.range" :style="{background: day.bgStyle.background}"></view>
					<view data-pointer="true" :style="{color: day.pointerStyle.color, background: day.pointerStyle.background}"><text>{{day.text}}</text></view>
					<view data-flag="true" :style="{background: day.flagStyle.background}"></view>
				</view>
			</view>
			<view>
				<view style="color: #999;">
					<block v-if="type=='rangetime'">
						<view class="mx-date-picker-press" @click="onSetTimePicker(true, 'begin')">开始时间：{{BeginDate}}<text :style="{color}">{{' '+BeginTime}}</text></view>
						<view class="mx-date-picker-press" @click="onSetTimePicker(true, 'end')">结束时间：{{EndDate}}<text :style="{color}">{{' '+EndTime}}</text></view>
					</block>
					<block v-else-if="type=='datetime'">
						<view class="mx-date-picker-press" @click="onSetTimePicker(true, 'begin')">当前选择：{{BeginDate}}<text :style="{color}">{{' '+BeginTime}}</text></view>
					</block>
					<block v-else-if="type=='range'">
						<view>开始日期：{{BeginDate}}</view>
						<view>结束日期：{{EndDate}}</view>
					</block>
					<block v-else-if="type=='date'">
						<view>当前选择：{{BeginDate}}</view>
					</block>
				</view>
				<view>
					<text class="mx-date-picker-press" @click="onCancel">取消</text>
					<text class="mx-date-picker-press" @click="onConfirm">确定</text>
				</view>
			</view>
		</view>
		<view v-if="showTimePicker||type=='time'" class="mx-date-picker">
			<view class="mx-date-picker-modal mx-date-picker-time">
				<view>
					<text>选择时间</text>
				</view>
				<view>
					<picker-view :value="timeValue" @change="onTimeChange">
						<picker-view-column><view v-for="(v,i) in 24" :key="i">{{i<10?'0'+i:i}}时</view></picker-view-column>
						<picker-view-column><view v-for="(v,i) in 60" :key="i">{{i<10?'0'+i:i}}分</view></picker-view-column>
						<picker-view-column v-if="showSeconds"><view v-for="(v,i) in 60" :key="i">{{i<10?'0'+i:i}}秒</view></picker-view-column>
					</picker-view>
				</view>
				<view>
					<view></view>
					<view>
						<text class="mx-date-picker-press" @click="onSetTimePicker(false)">取消</text>
						<text class="mx-date-picker-press" @click="onConfirmTime">确定</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			//颜色
			color: {
				type: String,
				default: '#409eff'
			},
			//是否显示秒 针对type为datetime或time时生效
			showSeconds: {
				type: Boolean,
				default: false
			},
			//初始的值
			value: [String, Array]
			,
			//类型date time datetime range rangetime
			type: {
				type: String,
				default: 'date'
			},
			//是否显示
			show: {
				type: Boolean,
				default: false
			},
			//初始格式
			format: {
				type: String,
				default: ''
			},
			//显示公历节日
			showHoliday: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				isShow: false,//是否显示
				date: {},
				weeks: ["一","二","三","四","五","六","日"],
				days: [],
				hackComputed: 0,
				checkedDateList: [],
				holidays: {'0101': '元旦','0214': '情人','0308': '妇女','0312': '植树','0401': '愚人','0501': '劳动','0504': '青年','0601': '儿童','0701': '建党','0801': '建军','0903': '抗日','0910': '教师','1001': '国庆','1031': '万圣','1224': '平安','1225': '圣诞'},
				showTimePicker: false,//是否显示时间选择器
				timeValue: [0, 0, 0],//时间选择器的值
				timeMode: 'begin',//begin:选择开始时间 end:选择结束时间
				beginTime: [0, 0, 0],//开始时间
				endTime: [0, 0, 0],//结束时间
				
			};
		},
		created(){
			this.setDefaultValue(this.value);
		},
		methods: {
			//转为标准日期格式
			parseDate(str){
				return new Date(str.replace('年','/').replace('月','/').replace('日','').replace(/-/g,'/'));
			},
			//比较日期（忽略时间）
			compareDate(a, b) {
				return (a.getMonth() == b.getMonth() && a.getFullYear() == b.getFullYear() && a.getDate() == b.getDate());
			},
			//优化时间数组
			formatTimeArray(time) {
				let tmp = [...time];
				if(!this.showSeconds) tmp.length = 2;
				tmp.forEach((v,k) => tmp[k] = ('0'+v).slice(-2) );
				return tmp.join(':');
			},
			//格式化日期
			formatDate(date, fmt) {
				var o = {
					"m+": date.getMonth() + 1, //月份   
					"d+": date.getDate(), //日   
					"h+": date.getHours(), //小时   
					"i+": date.getMinutes(), //分   
					"s+": date.getSeconds(), //秒   
					"q+": Math.floor((date.getMonth() + 3) / 3), //季度
				};
				if (/(y+)/.test(fmt))
					fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
				for (var k in o)
					if (new RegExp("(" + k + ")").test(fmt))
						fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
				return fmt;
			},
			//设定初始值
			setDefaultValue(value) {
				this.date = new Date();
				this.checkedDateList = [];
				if(value) {
					if(this.type=='time') {
						let arr = (''+value).split(':');
						if(arr && arr.length>1) {
							this.beginTime = [];
							arr.forEach(t => this.beginTime.push(parseInt(t)));
						} else {
							this.beginTime = [this.date.getHours(),this.date.getMinutes(),this.date.getSeconds()];
						}
						this.endTime = [...this.beginTime];
						this.timeValue = [...this.endTime];
					} else {
						if(this.type.indexOf('range')>=0) {//日期范围、日期时间范围
							if(Array.isArray(value) && value.length == 2) {
								value.forEach(date=> this.checkedDateList.push(this.parseDate(date)));
								this.checkedDateList.sort((a,b) => a - b);//从小到大排序
								this.beginTime = [this.checkedDateList[0].getHours(), this.checkedDateList[0].getMinutes(), this.checkedDateList[0].getSeconds()];
								this.endTime = [this.checkedDateList[1].getHours(), this.checkedDateList[1].getMinutes(), this.checkedDateList[1].getSeconds()];
								this.date = new Date(this.checkedDateList[0]);
							}
						} else {
							this.date = new Date(this.parseDate(value));
							this.checkedDateList = [this.date];
							this.beginTime = [this.date.getHours(),this.date.getMinutes(),this.date.getSeconds()];
						}
					}
				} else {
					this.checkedDateList = [this.date];
					this.beginTime = [this.date.getHours(),this.date.getMinutes(),this.date.getSeconds()];
					this.endTime = [...this.beginTime];
					this.timeValue = [...this.endTime];
				}
				this.refreshDays();
			},
			//刷新日期
			refreshDays() {
				let it = new Date(this.date);
				it.setDate(1);
				it.setDate(it.getDate() - ((it.getDay() == 0 ? 7 : it.getDay()) - 1));//偏移量
				this.days = [];
				for (let i = 0; i < 42; i++) {
					let day = {
						date: new Date(it),
						text: it.getDate(),
						checked: false,
						range: 'none',//none 正常 begin 开始 end 结束
						bgStyle: {},//背景样式
						pointerStyle: {},//焦点样式
						flagStyle: {}//标志样式
					};
					//选中了
					if(this.checkedDateList.find((value)=> this.compareDate(value,day.date) )){
						day.pointerStyle.background = this.color;
						day.pointerStyle.color = "#fff";
						day.checked = true;
					} else {
						let now = new Date();
						//今日
						if(this.compareDate(day.date,new Date())) {
							day.flagStyle.background = this.color;
						}
						//非本月的日期
						if(it.getMonth() < this.date.getMonth() || it.getMonth() > this.date.getMonth()) {
							day.pointerStyle.color = "#bbb";
						} else if(this.showHoliday) {
							//公历节日
							let holiday = this.formatDate(day.date, 'mmdd');
							if(this.holidays[holiday]) {
								day.text = this.holidays[holiday];
								day.flagStyle.background = this.color;
							}
						}
					}
					//范围选择
					if(this.checkedDateList.length==2) {
						let cur = +day.date, min = +this.checkedDateList[0], max = +this.checkedDateList[1];
						if(cur>=min && cur<=max) {
							day.bgStyle.background = this.color;
							if(this.compareDate(day.date, this.checkedDateList[0])) {
								day.range = 'begin';
							} else if(this.compareDate(day.date, this.checkedDateList[1])) {
								day.range = 'end';
							}
						}
					}
					this.days.push(day);
					it.setDate(it.getDate() + 1);
				}
			},
			//选择了日期
			onCheckedDay(day, index) {
				let checkedLength = this.checkedDateList.length;
				//当前选择的日期和已经选择的日期相等
				if(checkedLength && +day.date == +this.checkedDateList[0]) this.checkedDateList = [];
				if(this.type.indexOf('range') >=0) {
					if(checkedLength==2) this.checkedDateList = [];
				} else {
					this.checkedDateList = [];
				}
				this.checkedDateList.push(day.date);
				//从小到大排序
				this.checkedDateList.sort((a,b) => a - b);
				this.refreshDays();
			},
			//改变年份
			onSetYear(value) {
				this.date.setFullYear(this.date.getFullYear() + value);
				this.refreshDays();
				this.refreshComputed();
			},
			//改变月份
			onSetMonth(value) {
				this.date.setMonth(this.date.getMonth() + value);
				this.refreshDays();
				this.refreshComputed();
			},
			//设置时间选择器显示状态
			onSetTimePicker(show, mode) {
				if(show) {
					if(mode=='end' && this.checkedDateList.length != 2) return;
					this.timeMode = mode;
					if(this.timeMode=='begin') this.timeValue = this.beginTime;
					else if(this.timeMode=='end') this.timeValue =this.endTime;
				} else if(this.type=='time') {
					this.onCancel();
					return;
				}
				this.showTimePicker = show;
			},
			//确定时间选择
			onConfirmTime() {
				let index = -1;
				if(this.timeMode=='begin') this.beginTime = this.timeValue;
				else if(this.timeMode=='end') this.endTime =this.timeValue;
				this.refreshComputed();
				this.type!='time' ? this.onSetTimePicker(false) :this.onConfirm();
			},
			//时间选择变更
			onTimeChange(e) {
				this.timeValue = e.detail.value;
			},
			//取消
			onCancel() {
				this.$emit('cancel', false);
			},
			//确定
			onConfirm() {
				let result = {};
				if(this.type=='time') {
					result.value = this.formatTimeArray(this.beginTime);
				} else {
					//获取格式
					let format = this.format ? this.format : 'yyyy/mm/dd';
					//包含了时间
					if(this.type.indexOf('time')>=0) {
						//没有定义格式则需要追加时间格式进去
						if(!this.format) format += ' hh:ii' + (this.showSeconds ? ':ss' : '');
						//将选择的时间写进选择的日期里面
						let arr = [this.beginTime,this.endTime];
						this.checkedDateList.forEach((date,key)=>{
							date.setHours(arr[key][0]), date.setMinutes(arr[key][1]);
							if(this.showSeconds) date.setSeconds(arr[key][2]);
						});
					}
					//日期范围、日期时间范围
					if(this.type.indexOf('range')>=0) {
						if(this.checkedDateList.length!=2) return uni.showToast({icon:'none',title: "请选择两个日期"});
						result.value = [];
						this.checkedDateList.forEach(date=> result.value.push(this.formatDate(date, format)));
						result.date = [...this.checkedDateList];
					} else {//日期、日期时间
						result.value = this.formatDate(this.checkedDateList[0], format);
						result.date = new Date(this.checkedDateList[0]);
					}
				}
				this.$emit('confirm', result);
			},
			//强制更新computed
			refreshComputed(){
				this.hackComputed = +(new Date());
			}
		},
		computed: {
			Year() {
				this.hackComputed;
				return this.date.getFullYear();
			},
			Month() {
				this.hackComputed;
				return ('0' + (this.date.getMonth() + 1)).slice(-2);
			},
			BeginDate(){
				this.hackComputed;
				if(!this.checkedDateList.length) return '未选择';
				return this.formatDate(this.checkedDateList[0],'yyyy/mm/dd');
			},
			EndDate(){
				this.hackComputed;
				if(this.checkedDateList.length != 2) return '未选择';
				return this.formatDate(this.checkedDateList[1],'yyyy/mm/dd');
			},
			BeginTime(){
				this.hackComputed;
				return this.formatTimeArray(this.beginTime);
			},
			EndTime(){
				this.hackComputed;
				if(this.checkedDateList.length!=2) return '';
				return this.formatTimeArray(this.endTime);
			}
		},
		watch: {
			show(newValue, oldValue) {
				this.isShow = newValue;
			},
			value(newValue, oldValue) {
				this.setDefaultValue(newValue);
			}
		}
	}
</script>

<style lang="scss" scoped>
	$z-index: 100;
	$cell-spacing: 20upx;
	@keyframes MxDatePcikerModalFadeIn {
		from {opacity: 0;}
		to {opacity: 1;}
	}
	.mx-date-picker{
		position: fixed;
		z-index: $z-index;
		background: rgba(0,0,0,0.1);
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		font-size: 28upx;
		animation: MxDatePcikerModalFadeIn .2s 1;
		&-press{
			transition: 0.2s;
			&:active{
				background: #f5f5f5;
			}
		}
		&-time{
			width: 600upx !important;
			left: 75upx !important;
			picker-view{
				width: 100%;
				height: 198upx;
				line-height:66upx;
				text-align: center;
			}
		}
		&-modal{
			background: #fff;
			position: absolute;
			top: 50%;
			width: 700upx;
			left: 25upx;
			transform: translateY(-50%);
			box-shadow: 0 0 10px 0 rgba(0,0,0,0.2);
			border-radius: 12upx;
			>view:nth-child(1){
				text-align: center;
				line-height: 70upx;
				font-size: 32upx;
				.mx-date-picker-icon{
					display: inline-block;
					width: 70upx;
					height: 70upx;
					border-radius: 70upx;
					text-align: center;
					margin: 0 10upx;
				}
			}
			>view:nth-child(2){
				position: relative;
				display: flex;
				align-items: center;
				flex-wrap: wrap;
				>view{
					position: relative;
					width: 100upx;
					height: 100upx;
					line-height: 90upx;
					border-radius: 100upx;
					transition: .5s;
					>view{
						position: absolute;
						left: 5%;
						top: 5%;
						width: 90%;
						height: 90%;
						transition: .2s;
						&[data-bg=true]{opacity: .1;width: 100%;left: 0;}
						&[data-pointer=true]{border-radius: 100px;text-align: center;}
						&[data-flag=true]{position: absolute;left: 100%;top: 12upx;border-radius: 12upx;height: 12upx;width: 12upx;margin-left: -24upx;}
						&[data-range=begin]{border-top-left-radius: 100upx;border-bottom-left-radius: 100upx;width: 95%;left: 5%;}
						&[data-range=end]{border-top-right-radius: 100upx;border-bottom-right-radius: 100upx;width: 90%;}
					}
				}
			}
			>view:nth-child(3){
				padding: $cell-spacing;
				display: flex;
				justify-content: space-between;
				align-items: center;
				>view:last-child{
					text{
						padding: $cell-spacing*0.5 $cell-spacing;
						font-size: 32upx;
						&:first-child{
							color: #666;
						}
						&:last-child{
							color: #000;
						}
					}
				}
			}
		}
	}
	@font-face {
		font-family: "mxdatepickericon";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAMYAAsAAAAACBgAAALMAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDIgqDRIJiATYCJAMUCwwABCAFhG0HSRvfBsg+QCa3noNAyAQ9w6GDvbwpNp2vloCyn8bD/x+y+/5qDhtj+T4eRVEcbsCoKMFASzCgLdDkmqYDwgxkWQ6YH5L/YnppOlLEjlnter43YRjU7M6vJ3iGADVAgJn5kqjv/wEii23T86UsAQT+04fV+o97VTMx4PPZt4DlorLXwIQiGMA5uhaVrBWqGHfQXcTEiE+PE+g2SUlxWlLVBHwUYFMgrgwSB3wstTKSGzqF1nOyiGeeOtNjV4An/vvxR58PSc3AzrMViyDvPo/7dVEUzn5GROfIWAcU4rLXfMFdhte56y4We9gGNEVIezkBOOaQXUrbTf/hJVkhGpDdCw7dSOEzByMEn3kIic98hMxnAfeFPKWCbjRcA148/HxhCEkaA94eGWFaGolsblpaWz8/Po2WVuNHh1fmBpZHIpqal9fOjizhTteY+RZ9rv02I/pq0W6QVH3pSncBz3m55r9ZIPycHfmenvxe4uyutIgfT5u4bgkDusl9gcF0rnfnz+b2NpSaQWBFeu8GIL1xQj5AH/6FAsEr/50F28e/gA9ny6KjLrxIp0TE+UucmQOl5AFNLXkzZufWamWHYEI39PEP2If97CMdm51N6DSmIekwAVmneXTBr0PVYx+aTgfQbU3p+R4jKHdRurBq0oEw6AKSfm+QDbpGF/w3VOP+oBnMHbqdx409FjP4RRHHkAj5IWgQiBUjHfMTuQ1Icpg5avI4sQVRu8EHdWptM1aKrIjuscfeL+kZwxBTYoElztOQ2UygjRIjEphaZsyWodHgvm9SC8QC/JygEA6DiCDeEMhAQFhhOpvxa/18A0TiYMahIy0L2hYIZWeYH9JR085Al4qts1re5St2/SR6DINBGEVYQCWOETHDMAHZ+pcZIQJGTV4RtMmg8UbhuWL1+VLLA2RFHYC71kiRo0SNpjwQh8pj2EFU3oTNmS1WqgIA') format('woff2');
	}
	.mx-date-picker-icon {
	  font-family: "mxdatepickericon" !important;
	}
	.mx-date-picker-icon-you:before {
	  content: "\e63e";
	}
	.mx-date-picker-icon-zuo:before {
	  content: "\e640";
	}
	.mx-date-picker-icon-zuozuo:before {
	  content: "\e641";
	}
	.mx-date-picker-icon-youyou:before {
	  content: "\e642";
	}
</style>