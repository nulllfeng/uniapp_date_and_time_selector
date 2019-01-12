<template>
	<view>
		<view v-show="isShow" class="mask">
			<view class="modal">
				<view v-if="format!='time'" class="modal-header">
					<text class="modal-header-pointer" @click="onPrevMonth"><text class="icon icon-left"></text></text>
					<text>{{date.year}}年 - {{date.month}}月</text>
					<text class="modal-header-pointer" @click="onNextMonth"><text class="icon icon-right"></text></text>
					<view class="modal-header-colse" @click="onClose"><text class="icon icon-close"></text></view>
				</view>
				<view v-else class="modal-header">
					<text>选择时间</text>
					<view class="modal-header-colse" @click="onClose"><text class="icon icon-close"></text></view>
				</view>
				<view class="modal-body">
					<!-- 日期选择 -->
					<view v-if="format!='time'" class="date1">
						<view class="date1-header">
							<view v-for="(header,index) in ['一','二','三','四','五','六','日']" :key="index">{{header}}</view>
						</view>
						<view class="date1-body">
							<view v-for="(item,index) in dateList" :key="index" :id="index" @click="onSelectDate" :class="{'active': item.selected,'gray': item.is_gray}">{{item.date}}</view>
						</view>
					</view>
					<!-- 时间选择 -->
					<view v-if="format=='datetime' || format=='time'" class="time">
						<view v-if="format!='time'" class="line-title" style="padding: 10px 0;">选择时间</view>
						<view class="time-body">
							<view>
								<view :style="{transform: 'translateY('+timeObj.hour.y+'px)'}" id="hour" @touchstart="onTimeTouchStart"
								 @touchmove="onTimeTouchMove" @touchend="onTimeTouchEnd">
									<view v-for="value in 23+2">{{value-1>=0?(value-1<10?'0'+(value-1):value-1):'-'}}时</view>
								</view>
							</view>
							<view>
								<view :style="{transform: 'translateY('+timeObj.minute.y+'px)'}" id="minute" @touchstart="onTimeTouchStart"
								 @touchmove="onTimeTouchMove" @touchend="onTimeTouchEnd">
									<view v-for="value in 59+2">{{value-1>=0?(value-1<10?'0'+(value-1):value-1):'-'}}分</view>
								</view>
							</view>
							<view>
								<view :style="{transform: 'translateY('+timeObj.second.y+'px)'}" id="second" @touchstart="onTimeTouchStart"
								 @touchmove="onTimeTouchMove" @touchend="onTimeTouchEnd">
									<view v-for="value in 59+2">{{value-1>=0?(value-1<10?'0'+(value-1):value-1):'-'}}秒</view>
								</view>
							</view>
						</view>
					</view>
				</view>
				<view style="color: #666;font-size: 14px;padding: 10px 0;">
					已选择:
					<text v-if="format!='time'">
					{{date.year<10?('0'+date.year):date.year}}年
					{{date.month<10?('0'+date.month):date.month}}月
					{{date.date<10?('0'+date.date):date.date}}日
					</text>
					<text v-if="format=='datetime' || format=='time'">
					{{date.hour<10?('0'+date.hour):date.hour}}:
					{{date.minute<10?('0'+date.minute):date.minute}}:
					{{date.second<10?('0'+date.second):date.second}}
					</text>
				</view>
				<view>
					<view class="btn btn-primary" @click="onConfirm">确定</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			value: {
				type: Boolean,
				default: false
			},
			init: {
				type: String,
				default: ''
			},
			format: {
				type: String,
				default: 'datetime'
			}
		},
		data() {
			return {
				date: {
					year: 2019,
					month: 3,
					date: 1,
					hour: 0,
					minute: 0,
					second: 0
				},
				dateIndex: 0,
				dateList: [],
				timeObj: {
					hour: {
						num: 23 + 2,
						y: 0,
						old_y: 0,
						start_y: 0
					},
					minute: {
						num: 59 + 2,
						y: 0,
						old_y: 0,
						start_y: 0
					},
					second: {
						num: 59 + 2,
						y: 0,
						old_y: 0,
						start_y: 0
					}
				},
				isShow: false
			};
		},
		created(option) {},
		mounted() {
			if (this.init) {
				let initValue = this.init;
				if (this.format == 'time') initValue = '2019-1-1 ' + initValue;
				//转换 年-月-日 =>年/月/日
				let date = new Date(initValue.replace(/^(\d{4})-(\d{1,2})-(\d{1,2})$/, "$1/$2/$3"));
				this.date.year = date.getFullYear();
				this.date.month = date.getMonth() + 1;
				this.date.date = date.getDate();
				if (this.format.indexOf('time')) {
					this.date.hour = date.getHours();
					this.date.minute = date.getMinutes();
					this.date.second = date.getSeconds();
					this.timeObj.hour.y = date.getHours() * 20 * -1;
					this.timeObj.minute.y = date.getMinutes() * 20 * -1;
					this.timeObj.second.y = date.getSeconds() * 20 * -1;
				}
				if (this.format.indexOf('date') >= 0) {
					this.dateList = this.getDateList(this.date);
					this.dateList.forEach((item) => {
						if (item.year == this.date.year && item.month == this.date.month && item.date == this.date.date) {
							item.selected = true;
						}
					});
				}
			} else if (this.format.indexOf('date') >= 0) {
				this.dateList = this.getDateList(this.date);
			}
		},
		methods: {
			//关闭
			onClose() {
				this.isShow = false;
				this.$emit('input', this.isShow);
			},
			//确定
			onConfirm() {
				this.onClose();
				let date = JSON.parse(JSON.stringify(this.date));
				if (this.format == 'date') {
					date.value = date.year + '-' + date.month + '-' + date.date;
				} else if (this.format == 'datetime') {
					date.value = date.year + '-' + date.month + '-' + date.date + ' ' + date.hour + ':' + date.minute + ':' + date.second;
				} else {
					date.value = date.hour + ':' + date.minute + ':' + date.second;
				}
				this.$emit('selected', date);
			},
			//绑定时间拖动事件
			onTimeTouchStart(event) {
				let id = event.currentTarget.id;
				let cur = this.timeObj[id];
				cur.start_y = event.clientY, cur.old_y = cur.y;
			},
			//绑定时间拖动事件
			onTimeTouchMove(event) {
				let id = event.currentTarget.id;
				let cur = this.timeObj[id];
				cur.y = cur.old_y + event.clientY - cur.start_y;
			},
			//绑定时间拖动事件
			onTimeTouchEnd(event) {
				let id = event.currentTarget.id;
				let cur = this.timeObj[id];
				let itemHeight = 20,
					blockHeight = 60 - itemHeight;
				if ((cur.y * -1) < 0) {
					cur.y = 0;
				} else if ((cur.y * -1) > cur.num * itemHeight - blockHeight) {
					cur.y = -(cur.num * itemHeight - blockHeight);
				} else {
					cur.y = parseInt(cur.y / itemHeight) * itemHeight;
				}
				if (id == 'hour') this.date.hour = parseInt(cur.y / 30) * -1;
				if (id == 'minute') this.date.minute = parseInt(cur.y / 30) * -1;
				if (id == 'second') this.date.second = parseInt(cur.y / 30) * -1;
			},
			//上一月
			onPrevMonth() {
				this.date.month--;
				if (this.date.month <= 0) {
					this.date.year--;
					this.date.month = 12;
				}
				this.dateList = this.getDateList(this.date);
			},
			//下一月
			onNextMonth() {
				this.date.month++;
				if (this.date.month > 12) {
					this.date.year++;
					this.date.month = 1;
				}
				this.dateList = this.getDateList(this.date);
			},
			//选择日期
			onSelectDate(event) {
				let index = event.target.id;
				if (this.dateList[index].is_gray) return; //不能选择非本月日期
				for (let i = 0, len = this.dateList.length; i < len; i++) this.dateList[i].selected = false;
				this.dateList[index].selected = true;
				this.date.date = this.dateList[index].date;
			},
			//转换日期对象
			getDateInfo(date) {
				let weeks = ["日", "一", "二", "三", "四", "五", "六"];
				let data = {
					year: date.getFullYear(),
					month: date.getMonth() + 1,
					date: date.getDate(),
					week: weeks[date.getDay()],
					selected: false, //是否选中
					is_gray: false //是否显示灰色
				};
				data.text = `${data.year}-${data.month}-${data.date}`;
				return data;
			},
			//获取日期列表
			getDateList(option = {}) {
				let nowDate = new Date();
				let year = option.year || nowDate.getFullYear();
				let month = option.month ? option.month - 1 : nowDate.getMonth();
				let date = option.date || nowDate.getDate();
				let pointerDate = new Date();
				pointerDate.setFullYear(year);
				pointerDate.setMonth(month);
				pointerDate.setDate(1);
				let oneWeek = pointerDate.getDay() == 0 ? 7 : pointerDate.getDay(); //这月1号是周几
				pointerDate.setDate(pointerDate.getDate() - (oneWeek - 1));
				let dateList = [],
					is_selected = false;
				for (let i = 0; i < 42; i++) {
					let tmp = this.getDateInfo(pointerDate);
					tmp.is_gray = pointerDate.getMonth() < month || pointerDate.getMonth() > month; //设置非本月日期
					dateList.push(tmp);
					pointerDate.setDate(pointerDate.getDate() + 1);
				}
				return dateList;
			}
		},
		watch: {
			value(newVal) {
				this.isShow = newVal;
			}
		}
	}
</script>

<style>
	.mask {
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0);
		font-size: 16px;
		z-index: 1000;
	}

	.modal {
		background: #fff;
		position: absolute;
		left: 5%;
		right: 5%;
		top: 50%;
		transform: translateY(-50%);
		color: #606266;
		border-radius: 6px;
		box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.2);
		padding: 0 10px 10px 10px;
		box-sizing: border-box;
		text-align: center;
	}

	.modal-header {
		position: relative;
		padding: 10px 0;
		border-bottom: 1px solid #eee;
	}

	.modal-header-pointer {
		padding: 0 15px;
	}

	.modal-header-colse {
		position: absolute;
		right: 10px;
		top: 50%;
		transform: translateY(-50%);
	}

	.modal-body {
		padding: 10px 0;
	}

	.date1 {}


	.date1-header {
		position: relative;
		display: flex;
		justify-content: space-between;
		padding: 5px 0;
	}

	.date1-header view {
		display: block;
		width: 14.28%;
	}

	.date1-body {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		width: 100%;
	}

	.date1-body view {
		width: 14.28%;
		padding: 5px 0;
		transition: .2s;
		border-radius: 6px;
		text-align: center;
	}

	.date1-body view.active {
		background: #409eff;
		color: #fff;
	}

	.date1-body view.gray {
		background: none;
		color: #c0c4cc;
	}

	.time {}

	.time-body {
		display: flex;
		justify-content: space-between;
		position: relative;
	}

	.time-body>view {
		width: 33.33%;
		height: 60px;
		border-right: 1px solid #eee;
		overflow: hidden;
	}

	.time-body>view:last-child {
		border-right: none;
		width: 33.44%;
	}

	.time-body>view view {
		position: relative;
	}

	.time-body>view view view {
		height: 20px;
	}

	.time-body>view::before {
		content: "";
		position: absolute;
		top: 0;
		left: 0;
		height: 20px;
		width: 100%;
		background: linear-gradient(top, rgb(255, 255, 255), rgba(0, 0, 0, 0) 100%);
		z-index: 9;
		pointer-events: none;
	}

	.time-body>view::after {
		content: "";
		position: absolute;
		bottom: 0;
		left: 0;
		height: 20px;
		width: 100%;
		background: linear-gradient(bottom, rgb(255, 255, 255), rgba(0, 0, 0, 0) 100%);
		z-index: 9;
		pointer-events: none;
	}

	.line-title {
		position: relative;
		font-size: 14px;
		width: 100%;
		text-align: center;
		color: #666;
	}

	.line-title::before {
		content: "";
		display: inline-block;
		width: 30px;
		height: 1px;
		background: #ccc;
		position: relative;
		top: -4px;
		right: 6px;
	}

	.line-title::after {
		content: "";
		display: inline-block;
		width: 30px;
		height: 1px;
		background: #ccc;
		position: relative;
		top: -4px;
		right: -6px;
	}

	@font-face {
		font-family: "icondt";
		src: url('data:application/x-font-woff2;charset=utf-8;base64,d09GMgABAAAAAAL8AAsAAAAAB0QAAAKtAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHEIGVgCDFAqBfIFcATYCJAMQCwoABCAFhG0HUxtRBsiusCnDvUiJtoUQcQPnvVtvMMCnT/Hw/drrue/d3SConygCFR0eHy6CqjJVZVtlO74ZX0ugo/dyqp/OAdgfz4ZBXQtNLtsvRV7vk14dExXgwABohHp8A9bwLjsU6HnCWeSN0hVuWvowzYJJTfFg9rm/gkTApixA7f9/7/T/2tzJ57Vvua01aSx71Akw7AQU0FibIiuRgDyF2kUclLDyr44TGDSvqXF0cHYJCxIZF4gX1ihY6ISlDHL9QjuxNMVrjf7yslwBr/T3479+LFA0FZl6/Hw/ga1f/XqljtyVGMl5TgSrqNgAkng4mb5tWgSsaYP9Z2wBrRTKWcGuQ5OIc2370j9eUYkWKRwG23am8itOEPxCUPj1AhV+hra/m90BzlxqTxW0a29i05sRr6YTLTiDDCWc2Z2hFputH4mOOA8URaL5Hv51fMCPmzFRoiEZxKAG/j/UgibTxguO47b+ygEf64gwbJKoJTEjytLsBxVAd/1cTwE6NeqJZpavv2gguO5dXeG6fyjBFx+fFewkNx/0K8CSfysB6zKXEUtmKufiwpt6ULJgkMboIv5Jv2doZ+BSQr+ZHd4+cw1U/RbwpG5AY8g2tPrtwqB1h6uHTHGmRQ5gzZsFYVwNxahPUI17x5P6A41Zf9Aaj2kYdBlqyyFLYZeiIyWUYHSPNuZSM5ssmeJzMiFXri3TnGty3saw1+72c4dUkjtjhr8xfRGN2nGBB9ZplOeMleOUYmkPRapBp6On3tSOuYAlhxyiCJJAkXvIirGSls3F0uDz54gR5BS3gMyErxHHs52jnrYugB+qSwj5lle8G0afEBrSHFZAB5aT5DTNUDU9KEViom24I1kZ6DiNNKpu368tvnAbDCI3ZJSokdFongS+DevVqtLcc0i1VbjLfghsAgAAAAA=') format('woff2');
	}

	.icon {
		font-family: "icondt" !important;
		font-size: 16px;
		font-style: normal;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	.icon-close:before {
		content: "\e641";
	}

	.icon-right:before {
		content: "\e65f";
	}

	.icon-left:before {
		content: "\e660";
	}

	/*button*/
	.btn {
		display: block;
		line-height: 1;
		white-space: nowrap;
		cursor: pointer;
		background: #fff;
		border: 1px solid #dcdfe6;
		border-color: #dcdfe6;
		color: #606266;
		-webkit-appearance: none;
		text-align: center;
		box-sizing: border-box;
		outline: none;
		margin: 0;
		transition: .1s;
		font-weight: 500;
		-moz-user-select: none;
		-webkit-user-select: none;
		-ms-user-select: none;
		padding: 12px 20px;
		font-size: 14px;
		border-radius: 4px;
	}

	.btn-primary {
		color: #fff;
		background-color: #409eff;
		border-color: #409eff;
	}

	.btn-primary:active {
		background-color: #3A8EE6;
		border-color: #3A8EE6;
	}
</style>
