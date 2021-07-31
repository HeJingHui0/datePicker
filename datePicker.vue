<template>
  <div class="picker">
    <div class="picker-header">
      <div class="picker-header-fast" v-show="showHeaderFast">
        <div
          class="header-fast-item"
          :class="headerItemActive === index ? 'header-fast-active' : ''"
          v-for="(item, index) in fastChoose"
          :key="index"
          @click="handleHeaderItem(index)"
        >
          {{ item }}
        </div>
      </div>
      <div class="header-select-container" @click="changeTime">
        <div class="select-container-item">
          {{ startTitle }}
        </div>
        <div class="select-container-text">至</div>
        <div class="select-container-item">
          {{ endTitle }}
        </div>
      </div>
    </div>
    <div class="picker-box" :class="boxEnd ? 'picker-box-end' : ''" v-show="showPickerBox" tabindex="0" @blur="divBlur">
      <div class="picker-box-title">{{ flag ? '请选择结束日期' : '请选择开始日期' }}</div>
      <div class="picker-box-body">
        <div class="box-body-main">
          <div
            class="body-year"
            :class="yearActive ? 'year-active' : ''"
            @click="handleYear"
          >
            <span>{{ year }}年</span>
          </div>
          <div
            class="body-month"
            :class="monthActive ? 'month-active' : ''"
            @click="handleMonth"
          >
            <span>{{ month }}月</span>
          </div>
        </div>
        <div class="table-contianer" v-if="type === 'day'">
          <div class="table-contianer-header">
            <div class="contianer-header-item" v-for="(item, index) in teheadList" :key="index">{{item}}</div>
          </div>
          <div class="table-contianer-main">
            <div v-for="(item, index) in tbodyList" :key="index" class="contianer-main-item">
              <div class="contianer-main-subitem"
                :class="currentDay == subItem.month + '-' + subItem.name ? 'day-active' : ''"
                v-for="(subItem, subIndex) in item" :key="subIndex"
                @click="chooseDay(subItem)">
                <span v-if="subItem.name == 'default'"
                  :class="subItem.class" >
                  {{subItem.name}}
                </span>
                <span v-else
                  :class="subItem.class + (day == subItem.name ? ' currentMonth' : '')">
                  {{ subItem.name }}
                </span>
              </div>
            </div>
          </div>
        </div>
        <div class="time-container" v-else-if="type === 'year'">
          <div
            class="time-item"
            :class="item == year ? 'time-active' : ''"
            v-for="(item, index) in yearList"
            :key="index"
            @click="chooseYear(item)"
          >
            {{ item }}
          </div>
        </div>
        <div class="time-container" v-else>
          <div
            class="time-item"
            :class="item == month ? 'time-active' : ''"
            v-for="item in 12"
            @click="chooseMonth(item)"
          >
            {{ item }}月
          </div>
        </div>
        <div class="btn-container">
          <div class="location-btn" @click="locationTo()">定位到今天</div>
          <div class="clear-btn" @click="confirm">确定</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// author：HeJingHui0
// v-model绑定组件事件对象({startTime, endTime})
// showHeaderFast：是否显示快捷选择日期(默认显示)
// itemActive：快捷选择日期类型(-1：无(默认)；0：今日，1：昨日；2：本月；3：本季；4：本年；5：近一年)
// timeLimit：时间跨度(默认3年)
// handleTime：时间选择完触发的事件
export default {
  model:{ 
    prop: 'value',
    event: 'change',
  },
  props: {
    value: {
      type: Object,
      default:null
    },
    showHeaderFast: {
      type: Boolean,
      default: true
    },
    itemActive: {
      type: Number,
      default: -1
    },
    timeLimit: {
      type: Number,
      default: 3
    }
  },
  data: () => {
    return {
      teheadList: ['日', '一', '二', '三', '四', '五', '六'],
      year: '',
      month: 0,
      day: '',
      tbodyList: [],
      currentDay: '',
      type: 'day',
      yearList: [],
      yearActive: false,
      monthActive: false,
      fastChoose: ['今日', '昨日', '本月', '本季', '本年', '近一年'],
      startTime: '',
      endTime: '',
      startTitle: '开始日期',
      endTitle: '结束日期',
      showPickerBox: false,
      flag: -1,
      headerItemActive: -1,
      useFast: true,
      boxEnd: false
    }
  },
  watch: {
    month() {
      if (this.month < 1) {
        this.month = 12
        this.year--
      } else if (this.month > 12) {
        this.month = 1
        this.year++
      }
      this.tbodyList = []
      this.handleAetpicker()
    },
    itemActive: {
      handler() {
        this.headerItemActive = this.itemActive
        if(this.headerItemActive != -1) {
          this.handleHeaderItem(this.itemActive)
        }
      },
      deep: true,
      immediate: true
    },
    showPickerBox: {
      handler() {
        if(this.showPickerBox) {
          this.getInitTime()
        }
      },
      immediate: true
    }
  },
  mounted() {
    this.getInitTime()
  },
  methods: {
    handleYear() {
      this.type = 'year'
      this.yearActive = true
      this.monthActive = false
    },
    handleMonth() {
      this.type = 'month'
      this.monthActive = true
      this.yearActive = false
    },
    chooseYear(i) {
      this.type = 'month'
      this.year = i
      this.monthActive = true
      this.yearActive = false
    },
    chooseMonth(i) {
      this.type = 'day'
      this.month = i
      this.monthActive = false
      this.yearActive = false
    },
    chooseDay(i) {
      this.currentDay = `${i.month}-${i.name}`
      this.day = i.name
    },
    handleHeaderItem(index, isFirst = false) {
      if (!isFirst) {
        this.headerItemActive = index
        this.useFast = true
        this.startTime = this.getTime(index)[0]
        this.endTime = this.getTime(index)[1]
        let startYear = new Date(this.startTime).getFullYear()
        let startMonth = new Date(this.startTime).getMonth() + 1
        let startDay = new Date(this.startTime).getDate()
        let endYear = new Date(this.endTime).getFullYear()
        let endMonth = new Date(this.endTime).getMonth() + 1
        let endDay = new Date(this.endTime).getDate()
        this.startTitle = `${startYear}-${this.formatTime(
          startMonth
        )}-${this.formatTime(startDay)}`
        this.endTitle = `${endYear}-${this.formatTime(
          endMonth
        )}-${this.formatTime(endDay)}`
        this.$emit('change', {startTime: this.startTime, endTime: this.endTime})
        this.$emit('handleTime')
      }
    },
    formatTime(time) {
      return time < 10 ? `0${time}` : time
    },
    getTime(type) {
      let date = new Date()
      let year = date.getFullYear()
      let month = date.getMonth()
      let day = date.getDate()
      let result = []
      if (type === 0) {
        result[0] = new Date(year, month, day).getTime()
        result[1] = new Date(year, month, day + 1).getTime() - 1
      } else if (type === 1) {
        result[0] = new Date(year, month, day - 1).getTime()
        result[1] = new Date(year, month, day).getTime() - 1
      } else if (type === 2) {
        result[0] = new Date(year, month, 1).getTime()
        result[1] = new Date(year, month + 1, 1).getTime() - 1
      } else if (type === 3) {
        if (month >= 0 && month <= 2) {
          result[0] = new Date(year, 0, 1).getTime()
          result[1] = new Date(year, 3, 1).getTime() - 1
        } else if (month >= 3 && month <= 5) {
          result[0] = new Date(year, 3, 1).getTime()
          result[1] = new Date(year, 6, 1).getTime() - 1
        } else if (month >= 6 && month <= 8) {
          result[0] = new Date(year, 6, 1).getTime()
          result[1] = new Date(year, 9, 1).getTime() - 1
        } else {
          result[0] = new Date(year, 9, 1).getTime()
          result[1] = new Date(year + 1, 0, 1).getTime() - 1
        }
      } else if (type === 4) {
        result[0] = new Date(year, 0, 1).getTime()
        result[1] = new Date(year + 1, 0, 1).getTime() - 1
      } else if (type === 5) {
        result[0] = new Date(year - 1, month, day).getTime()
        result[1] = new Date(year, month, day).getTime()
      }
      return result
    },
    divBlur() {
      if (!this.endTime) {
        this.startTime = ''
        this.startTitle = '开始日期'
        this.endTime = ''
        this.endTitle = '结束日期'
        this.getInitTime()
      }
      this.flag = -1
      this.boxEnd = false
      this.showPickerBox = false
      this.$emit('change', {startTime: this.startTime, endTime: this.endTime})
      this.$emit('handleTime')
    },
    confirm() {
      let time = new Date(`${this.year}-${this.month}-${this.day}`).getTime()
      if (`${this.month}-${this.day}` === this.currentDay) {
        if (!this.flag) {
          this.boxEnd = true
          let limit = new Date(`${this.year + this.timeLimit}-${this.month}-${this.day}`).getTime()
          if (this.endTime && this.endTime > limit) {
            this.$message.warning(`时间跨度不允许超过${this.timeLimit}年`)
            return
          } else if (this.endTime && this.endTime < time) {
            this.$message.warning('开始日期不能大于结束日期')
            return
          }
          this.startTitle = `${this.year}-${this.formatTime(
            this.month
          )}-${this.formatTime(this.day)}`
          this.startTime = time
        } else {
          let limit = new Date(`${this.year - this.timeLimit}-${this.month}-${this.day}`).getTime()
          if (this.startTime && this.startTime < limit) {
            this.$message.warning(`时间跨度不允许超过${this.timeLimit}年`)
            return
          } else if (this.startTime && this.startTime > time) {
            this.$message.warning('结束日期不能小于开始日期')
            return
          }
          this.endTitle = `${this.year}-${this.formatTime(
            this.month
          )}-${this.formatTime(this.day)}`
          this.endTime = time + 86399999
        }
        this.headerItemActive = -1
        this.getInitTime(true)
        if (!this.flag) {
          this.flag ++
        } else {
          this.boxEnd = false
          this.showPickerBox = false
          this.flag = -1
          this.$emit('change', {startTime: this.startTime, endTime: this.endTime})
          this.$emit('handleTime')
        }
        
      }
    },
    locationTo() {
      this.getInitTime(true)
      this.type = 'day'
      this.monthActive = false
      this.yearActive = false
    },
    changeTime() {
      if(this.useFast) {
        this.startTime = ''
        this.endTime = ''
      }
      this.useFast = false
      this.showPickerBox = true
      this.flag ++
      this.$nextTick(() => {
        document.getElementsByClassName('picker-box')[0].focus()
      })
    },
    getInitTime(flag = false) {
      this.yearList = []
      let date = new Date()
      this.year = date.getFullYear()
      this.month = date.getMonth() + 1
      this.day = date.getDate()
      this.currentDay = `${this.month}-${this.day}`
      for (let i = -4; i < 8; i++) {
        this.yearList.push(this.year + i)
      }
      if (!flag) {
        this.startTime = this.getTime(this.headerItemActive)[0]
        this.endTime = this.getTime(this.headerItemActive)[1]
        this.handleHeaderItem(this.itemActive, true)
      }
      this.handleAetpicker()
    },
    // 生成档期数据
    handleAetpicker() {
      this.tbodyList = []
      let days = new Date(this.year, this.month, 0).getDate()
      let week = new Date(this.year, this.month - 1, 1).getDay()
      let last_month = new Date(this.year, this.month - 1, 0).getDate()
      this.tbodyList[0] = []
      for (let i = 0; i < Math.ceil((days + week) / 7) * 7; i++) {
        let nub = Math.floor(i / 7)
        if (i < week) {
          this.tbodyList[nub].push({
            class: 'default',
            name: last_month + i - week + 1,
            month: this.month == 0 ? 12 : this.month - 1
          })
        } else {
          if (!this.tbodyList[nub]) {
            this.tbodyList[nub] = []
          }
          let day = i - week + 1
          let calssName = 'actives'
          let month = this.month
          if (day > days) {
            day -= days
            calssName = 'default'
            month = this.month + 1 > 12 ? 1 : this.month + 1
          }
          this.tbodyList[nub].push({
            class: calssName,
            name: day,
            month: month,
          })
        }
      }
      let arr = this.tbodyList[Math.floor((week + days) / 7)]
      if (arr && arr.length !== 7) {
        this.tbodyList[Math.floor((week + days) / 7)] = arr.concat(
          new Array(7 - arr.length).fill('')
        )
      }
    },
  },
}
</script>

<style lang="less" scoped>
.picker {
  position: relative;
  height: 29px;
  vertical-align: top;
  .picker-header {
    display: flex;
    height: 29px;
    .picker-header-fast {
      display: flex;
      border: 1px solid #becbd9;
      border-radius: 4px;
      background-color: #ffffff;
      margin-right: 10px;
      cursor: pointer;
      .header-fast-item {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 60px;
        height: 27px;
        line-height: 27px;
        font-size: 6px;
        color: #97a7be;
        border-right: 1px solid #becbd9;
      }
      .header-fast-active {
        color: #ffffff;
        background-color: #3b85d6;
      }
      .header-fast-item:last-child {
        border-right: none;
      }
    }
    .header-select-container {
      display: flex;
      align-items: center;
      background-color: #ffffff;
      margin-right: 10px;
      border: 1px solid #becbd9;
      border-radius: 4px;
      width: 300px;
      height: 27px;
      line-height: 27px;
      .select-container-item {
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 7px;
        color: #97a7be;
        width: 144px;
        cursor: pointer;
        height: 20px;
        line-height: 20px;
      }
      .select-container-text {
        font-size: 6px;
        width: 12px;
        color: #becbd9;
        height: 20px;
        line-height: 20px;
      }
    }
  }
  .picker-box {
    position: absolute;
    top: 40px;
    left: 377px;
    width: 300px;
    height: 345px;
    background-color: #ffffff;
    cursor: pointer;
    padding-top: 15px;
    z-index: 999;
    border: 1px solid #ccc;
    border-radius: 4px;
    outline: none;
    .picker-box-title {
      font-size: 16px;
      font-weight: bold;
      height: 20px;
      line-height: 20px;
      color: #515151;
      margin-left: 15px;
    }
    .picker-box-body {
      .box-body-main {
        display: flex;
        margin: 10px 0 5px 15px;
        .body-year {
          font-size: 6px;
          color: #515151;
          width: 85px;
          height: 32px;
          background-color: #f3f3f3;
          margin-right: 10px;
          display: flex;
          justify-content: center;
          align-items: center;
          border-radius: 2px;
          img {
            display: inline-block;
            width: 16px;
            height: 16px;
            margin-left: 8px;
          }
        }
        .body-month {
          font-size: 6px;
          color: #515151;
          width: 54px;
          height: 32px;
          background-color: #f3f3f3;
          display: flex;
          justify-content: center;
          align-items: center;
          border-radius: 2px;
        }
        .year-active {
          border-bottom: 2px solid #3b85d6;
        }
        .month-active {
          border-bottom: 2px solid #3b85d6;
        }
      }
      .table-contianer {
        display: block;
        font-size: 7px;
        height: 220px;
        color: #515151;
        .table-contianer-header {
          display: flex;
          justify-content: center;
          align-items: center;
          .contianer-header-item {
            display: flex;
            justify-content: center;
            align-items: center;
            width: 40px;
            height: 36px;
          }
        }
        .table-contianer-main {
          overflow-y: auto;
          height: 180px;
          .contianer-main-item {
            display: flex;
            justify-content: center;
            align-items: center;
            .contianer-main-subitem {
              display: flex;
              justify-content: center;
              align-items: center;
              width: 40px;
              height: 36px;
              .default {
                opacity: 0.5;
              }
            }
            .contianer-main-subitem:hover {
              background-color: #eef1f6;
              border-radius: 2px;
            }
            .day-active {
              background-color: #3b85d6 !important;
              color: #ffffff;
              border-radius: 2px;
            }
          }
        }
      }
      .time-container {
        display: flex;
        flex-wrap: wrap;
        padding: 0 14px;
        height: 218px;
        .time-item {
          font-size: 7px;
          display: flex;
          justify-content: center;
          align-items: center;
          width: 60px;
          height: 30px;
          margin-right: 46px;
          margin-top: 24px;
        }
        .time-item:hover {
          background-color: #eef1f6;
          border-radius: 2px;
        }
        .time-item:nth-child(3n) {
          margin-right: 0;
        }
        .time-active {
          background-color: #3b85d6 !important;
          border-radius: 2px;
          color: #ffffff;
        }
      }
      .btn-container {
        display: flex;
        margin-top: 10px;
        border-top: 1px solid #d4d4d4;
        font-size: 6px;
        color: #3b85d6;
        position: relative;
        .clear-btn {
          position: absolute;
          top: 10px;
          right: 20px;
          border: 1px solid #ccc;
          border-radius: 4px;
          padding: 5px 10px;
        }
        .location-btn {
          position: absolute;
          top: 10px;
          right: 80px;
          border: 1px solid #ccc;
          border-radius: 4px;
          padding: 5px 10px;
        }
      }
    }
  }
  .picker-box-end {
    left: 425px;
  }
}
</style>