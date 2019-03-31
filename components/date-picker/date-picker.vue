<template>
	<view class="picker">
		<view class="picker-modal">
			<view class="picker-modal-header">
				<text class="picker-icon picker-icon-zuozuo" @click="onSetYear(-1)"></text>
				<text class="picker-icon picker-icon-zuo" @click="onSetMonth(-1)"></text>
				<text>{{year}}年{{month}}月</text>
				<text class="picker-icon picker-icon-you" @click="onSetMonth(+1)"></text>
				<text class="picker-icon picker-icon-youyou" @click="onSetYear(+1)"></text>
			</view>
			<swiper class="picker-modal-body" :indicator-dots="false" :autoplay="false" :circular="true" :duration="200" :current="calendarIndex"
			 @change="onSwiperChange">
				<swiper-item class="picker-calendar" v-for="(calendar,calendarIndex2) in calendars" :key="calendarIndex2">
					<view class="picker-calendar-view" v-for="(week,index) in weeks" :key="index - 7">
						<view>{{week}}</view>
					</view>
					<view class="picker-calendar-view" v-for="(date,dateIndex) in calendar" :key="dateIndex" @click="onSelectDate(date)">
						<view data-pointer="true" :style="{color: date.pointerStyle.color, background: date.pointerStyle.background}">{{date.text}}</view>
					</view>
				</swiper-item>
			</swiper>
			<view class="picker-modal-footer">
			</view>
		</view>
	</view>
</template>

<script>
	//转为标准日期格式
	const parseDate = s => new Date(s.replace(/(年|月|-)/g, '/').replace(/(日)/g, ''));
	//比较日期（忽略时间）
	const compareDate = (a, b) => a.getMonth() == b.getMonth() && a.getFullYear() == b.getFullYear() && a.getDate() == b.getDate();
	//生成日历
	const genCalendar = (date, proc) => {
		let it = new Date(date),
			calendars = [];
		it.setDate(1);
		it.setDate(it.getDate() - ((it.getDay() == 0 ? 7 : it.getDay()) - 1)); //偏移量
		for (let i = 0; i < 42; i++) {
			let tmp = {
				dateObj: new Date(it),
				text: it.getDate(),
				gray: it.getMonth() < date.getMonth() || it.getMonth() > date.getMonth()
			};
			calendars.push(Object.assign(tmp, proc ? proc(tmp) : {}));
			it.setDate(it.getDate() + 1);
		}
		return calendars;
	};
	//获取日期到指定的月份
	const getDate = (d, v) => {
		let dd = new Date(d);
		dd.setMonth(dd.getMonth() + v, 1);
		return dd;
	};

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
			value: [String, Array],
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
				isShow: false, //是否显示
				date: new Date(),
				weeks: ["一", "二", "三", "四", "五", "六", "日"],
				days: [],
				hackComputed: 0,
				checkedDateList: [],
				holidays: {
					'0101': '元旦',
					'0214': '情人',
					'0308': '妇女',
					'0312': '植树',
					'0401': '愚人',
					'0501': '劳动',
					'0504': '青年',
					'0601': '儿童',
					'0701': '建党',
					'0801': '建军',
					'0903': '抗日',
					'0910': '教师',
					'1001': '国庆',
					'1031': '万圣',
					'1224': '平安',
					'1225': '圣诞'
				},
				showTimePicker: false, //是否显示时间选择器
				timeValue: [0, 0, 0], //时间选择器的值
				timeMode: 'begin', //begin:选择开始时间 end:选择结束时间
				beginTime: [0, 0, 0], //开始时间
				endTime: [0, 0, 0], //结束时间
				/*-------------*/
				calendars: [],
				calendarIndex: 1,
				year: '2019',
				month: '3',
				checkeds: [],//选中的集合

			};
		},
		created() {
			this.refreshCalendars(true);
		},
		methods: {
			//改变年份
			onSetYear(value) {
				this.date.setFullYear(this.date.getFullYear() + value);
				this.refreshCalendars(true);
			},
			//改变月份
			onSetMonth(value) {
				this.date.setMonth(this.date.getMonth() + value);
				this.refreshCalendars(true);
			},
			//获取日历
			getCalendar(date) {
				return genCalendar(date, param => {
					param.pointerStyle = {
						color: param.gray ? '#999' : '#333',
						background: 'transparent'
					};
					this.checkeds.forEach(date=>{
						if(!param.gray && compareDate(date,param.dateObj)) {
							param.pointerStyle.background = this.color;
							param.pointerStyle.color = '#fff';
						}
					});
					return param;
				});
			},
			//刷新日历
			refreshCalendars(init = false) {
				let date = new Date(this.date);
				let before = getDate(date, -1);
				let after = getDate(date, +1);
				if (init) {
					this.calendars = [this.getCalendar(before), this.getCalendar(date), this.getCalendar(after)];
				} else {
					if (this.calendarIndex == 0) {
						this.calendars.splice(0, 1, this.getCalendar(date));
						this.calendars.splice(1, 1, this.getCalendar(after));
						this.calendars.splice(2, 1, this.getCalendar(before));
					} else if (this.calendarIndex == 1) {
						this.calendars.splice(0, 1, this.getCalendar(before));
						this.calendars.splice(1, 1, this.getCalendar(date));
						this.calendars.splice(2, 1, this.getCalendar(after));
					} else if (this.calendarIndex == 2) {
						this.calendars.splice(0, 1, this.getCalendar(after));
						this.calendars.splice(1, 1, this.getCalendar(before));
						this.calendars.splice(2, 1, this.getCalendar(date));
					}
				}
				this.year = this.date.getFullYear();
				this.month = ('0' + (this.date.getMonth() + 1)).slice(-2);
			},
			//滑块切换
			onSwiperChange(e) {
				this.calendarIndex = e.detail.current;
				let calendar = this.calendars[this.calendarIndex];
				this.date = new Date(calendar[22].dateObj);//取中间一天，保证是当前的月份
				this.refreshCalendars();
			},
			//选中日期
			onSelectDate(date) {
				if(date.gray) return false;
				if(this.checkeds.length) this.checkeds = [];
				this.checkeds.push(date.dateObj);
				this.refreshCalendars();
			}
		}
	}
</script>

<style lang="scss" scoped>
	$z-index: 100;
	$cell-spacing: 20upx;

	@keyframes MxDatePcikerModalFadeIn {
		from {
			opacity: 0;
		}

		to {
			opacity: 1;
		}
	}

	.picker {
		position: fixed;
		z-index: $z-index;
		background: rgba(0, 0, 0, 0.1);
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		font-size: 28upx;
		animation: MxDatePcikerModalFadeIn .2s 1;
		&-modal {
			background: #fff;
			position: absolute;
			top: 50%;
			left: 25upx;
			width: 700upx;
			transform: translateY(-50%);
			box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.1);
			border-radius: 12upx;

			&-header {
				text-align: center;
				line-height: 70upx;
				font-size: 32upx;

				.picker-icon {
					display: inline-block;
					width: 70upx;
					height: 70upx;
					border-radius: 70upx;
					text-align: center;
					margin: 0 10upx;
				}
			}

			&-body {
				width: 700upx;
				height: 700upx;
				position: relative;
			}
		}

		&-calendar {
			width: 100%;
			height: 100%;
			position: relative;
			display: flex;
			align-items: center;
			flex-wrap: wrap;
			&-view {
				position: relative;
				width: 100upx;
				height: 100upx;
				>view{
					position: absolute;
					width: 80upx;
					height: 80upx;
					line-height: 80upx;
					border-radius: 80upx;
					text-align: center;
					left: 50%;
					top: 50%;
					transform: translate(-50%,-50%);
					transition: .3s;
				}
			}
		}
	}

	@font-face {
		font-family: "mxdatepickericon";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAMYAAsAAAAACBgAAALMAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDIgqDRIJiATYCJAMUCwwABCAFhG0HSRvfBsg+QCa3noNAyAQ9w6GDvbwpNp2vloCyn8bD/x+y+/5qDhtj+T4eRVEcbsCoKMFASzCgLdDkmqYDwgxkWQ6YH5L/YnppOlLEjlnter43YRjU7M6vJ3iGADVAgJn5kqjv/wEii23T86UsAQT+04fV+o97VTMx4PPZt4DlorLXwIQiGMA5uhaVrBWqGHfQXcTEiE+PE+g2SUlxWlLVBHwUYFMgrgwSB3wstTKSGzqF1nOyiGeeOtNjV4An/vvxR58PSc3AzrMViyDvPo/7dVEUzn5GROfIWAcU4rLXfMFdhte56y4We9gGNEVIezkBOOaQXUrbTf/hJVkhGpDdCw7dSOEzByMEn3kIic98hMxnAfeFPKWCbjRcA148/HxhCEkaA94eGWFaGolsblpaWz8/Po2WVuNHh1fmBpZHIpqal9fOjizhTteY+RZ9rv02I/pq0W6QVH3pSncBz3m55r9ZIPycHfmenvxe4uyutIgfT5u4bgkDusl9gcF0rnfnz+b2NpSaQWBFeu8GIL1xQj5AH/6FAsEr/50F28e/gA9ny6KjLrxIp0TE+UucmQOl5AFNLXkzZufWamWHYEI39PEP2If97CMdm51N6DSmIekwAVmneXTBr0PVYx+aTgfQbU3p+R4jKHdRurBq0oEw6AKSfm+QDbpGF/w3VOP+oBnMHbqdx409FjP4RRHHkAj5IWgQiBUjHfMTuQ1Icpg5avI4sQVRu8EHdWptM1aKrIjuscfeL+kZwxBTYoElztOQ2UygjRIjEphaZsyWodHgvm9SC8QC/JygEA6DiCDeEMhAQFhhOpvxa/18A0TiYMahIy0L2hYIZWeYH9JR085Al4qts1re5St2/SR6DINBGEVYQCWOETHDMAHZ+pcZIQJGTV4RtMmg8UbhuWL1+VLLA2RFHYC71kiRo0SNpjwQh8pj2EFU3oTNmS1WqgIA') format('woff2');
	}

	.picker-icon {
		font-family: "mxdatepickericon" !important;
	}

	.picker-icon-you:before {
		content: "\e63e";
	}

	.picker-icon-zuo:before {
		content: "\e640";
	}

	.picker-icon-zuozuo:before {
		content: "\e641";
	}

	.picker-icon-youyou:before {
		content: "\e642";
	}
</style>
