<template>
  <view class="container">
    <uni-calendar :insert="true" :lunar="true" :date="selectedDay" :selected="clockInArr" @change="change"
      :key="calendarKey" />
    <view style="margin-top: 16px;">
      <button type="primary" @click="clockInOrCancel">{{status ? '取消打卡' : '打卡'}}</button>
    </view>
  </view>
</template>

<script>
  /**
   * 获取任意时间
   */
  function getDate(date, AddDayCount = 0) {
    if (!date) {
      date = new Date()
    }
    if (typeof date !== 'object') {
      date = date.replace(/-/g, '/')
    }
    const dd = new Date(date)

    dd.setDate(dd.getDate() + AddDayCount) // 获取AddDayCount天后的日期

    const y = dd.getFullYear()
    const m = dd.getMonth() + 1 < 10 ? '0' + (dd.getMonth() + 1) : dd.getMonth() + 1 // 获取当前月份的日期，不足10补0
    const d = dd.getDate() < 10 ? '0' + dd.getDate() : dd.getDate() // 获取当前几号，不足10补0
    return {
      fulldate: y + '-' + m + '-' + d,
      year: y,
      month: m,
      date: d,
      day: dd.getDay()
    }
  }

  export default {
    data() {
      return {
        selectedDay: '',
        status: false,
        clockInArr: [],
        calendarKey: 0
      }
    },
    mounted() {
      // #ifdef WEB
      window.home = this;
      // #endif
      this.selectedDay = getDate().fulldate;
      
      uni.getStorage({
        key: 'arr',
        success: res => {
          // console.log(res.data);
          this.clockInArr = JSON.parse(res.data);
        }
      })
    },
    methods: {
      change(e) {
        this.selectedDay = e.fulldate;
        this.status = this.clockInArr.some(item => item.date === e.fulldate);
      },
      clockInOrCancel() {
        if (this.status) {
          const idx = this.clockInArr.findIndex(item => item.date === this.selectedDay);
          if (idx > -1) {
            this.clockInArr.splice(idx, 1);
            this.status = false;
          }
        } else {
          this.clockInArr.push({
            date: this.selectedDay,
            info: "打卡"
          });
          this.status = true;
        }

        uni.setStorage({
          key: 'arr',
          data: JSON.stringify(this.clockInArr),
          success: () => {}
        })
        // 手动触发日历组件的更新
        this.calendarKey++;
      }
    }
  }
</script>

<style>
  .container {
    padding: 20px;
    font-size: 14px;
    line-height: 24px;
  }
</style>