<template>
  <div class="calendar">
    <div class="contant">
      <div class="left">
        <div  class="inputdate">
          <el-input v-model="inputDate"  placeholder="YYYY-MM-DD"></el-input>
          <el-input v-model="daytime"></el-input>
        </div>
        <div class="header">
          <div class="icon-left">
            <i  class="el-icon-d-arrow-left" @click="prevYear"></i>
            <i class="el-icon-arrow-left" @click="prevMonth"></i>
          </div>
          <div class="date-content">  {{ currentYear }} 年 {{ currentMonth }} </div>
          <div>
        <i class="el-icon-arrow-right" @click="nextMonth"></i>
        <i class="el-icon-d-arrow-right" @click="nextYear"></i>
      </div>
        </div>
        <table>
          <thead>
            <tr>
              <th v-for="day in daysOfWeek" :key="day">{{ day }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(week, i) in weeks" :key="i">
              <td v-for="(date, j) in week" :key="j" :class="{ disabled: !isDateEnabled(date), empty: date === null,'not-current-month':!isCurrentMonth(date) ,'selected':isSelected(date)}"
                @click="selectDate(date)">
                {{ date && date.getDate() }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  </div>
  </div>
</template>

<script>
export default {
  name: 'Calendar',
  props: {
    disabledStartDate: Date,
    disabledEndDate: Date,
    selectedDate: Date,
  },
  data() {
    return {
      currentDate: new Date(),
      daysOfWeek: ['日', '一', '二', '三', '四', '五', '六'],
      inputDate: '',
    };
  },
  watch: {
    inputDate(newDate) {
  const datePattern = /^(\d{4})-(\d{2})-(\d{2})$/;
  const match = newDate.match(datePattern);

  if (match) {
    const year = parseInt(match[1]);
    const month = parseInt(match[2]) - 1; 
    const day = parseInt(match[3]);

    const parsedDate = new Date(year, month, day);

  
    if (parsedDate.getFullYear() === year && parsedDate.getMonth() === month && parsedDate.getDate() === day) {
      if (this.isDateEnabled(parsedDate)) {
        this.currentDate.setFullYear(parsedDate.getFullYear());
        this.currentDate.setMonth(parsedDate.getMonth());
        this.currentDate.setDate(parsedDate.getDate());
        this.currentDate = new Date(this.currentDate);
        this.$emit('updateselectedDate', parsedDate); 
      }
    }
  }

},
  },
  computed: {

    currentMonth() {
      return this.currentDate.toLocaleString('default', { month: 'long' });
    },
    currentYear() {
      return this.currentDate.getFullYear();
    },
    daytime() {
      return this.selectedDate.toLocaleString('default', { weekday: 'long' });
    },
    //获取当天的日期
    currentDay() {
      return this.currentDate.getDate();
    },
    weeks() {
      let weeks = [];
      const firstDayOfMonth = new Date( //获取当前月份的第一天
        this.currentDate.getFullYear(),
        this.currentDate.getMonth(),
      );
      const lastDayOfMonth = new Date( //获取当前月份的最后一天
        this.currentDate.getFullYear(),
        this.currentDate.getMonth() + 1,
        0
      );
      let currentWeek = Array(firstDayOfMonth.getDay()).fill(null);//获取当前月份第一天是星期几
      let currentDate = firstDayOfMonth;//获取当前月份的第一天
      //获得上个月的数据
      if (currentWeek.length > 0) {
        const prevMonth = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() - 1, 0);
        for (let i = 0; i < currentWeek.length; i++) {
          currentWeek[i] = new Date(prevMonth.getFullYear(), prevMonth.getMonth(), prevMonth.getDate() - i);
        }
        currentWeek = currentWeek.reverse();
      }

      while (currentDate <= lastDayOfMonth) {//循环获取当前月份的所有日期
        currentWeek.push(new Date(currentDate));
        currentDate.setDate(currentDate.getDate() + 1);

        if (currentWeek.length === 7 || currentDate > lastDayOfMonth) { //如果当前日期是星期六或者当前日期大于当前月份的最后一天
          let nextMonthDate = 1;
          while (currentWeek.length < 7) {
            currentWeek.push(new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() +1,nextMonthDate));
            nextMonthDate++;
          }
          weeks.push(currentWeek);
          currentWeek = [];
        }
      }
      return weeks;
    },
  },
  methods: {
    isCurrentMonth(date) {
      if(!date)return false;
      return (
       date.getFullYear() === this.currentDate.getFullYear()
        && date.getMonth() === this.currentDate.getMonth()
      )
    },
    updateInputDate(date) {
      this.inputDate = date.getFullYear()+'-'+('0'+(date.getMonth()+1)).slice(-2)+'-'+('0'+date.getDate()).slice(-2);
    },
    isDateEnabled(date) { //检查当前日期是否可选
    // let flag =  isCurrentMonth(date)
      return (
        date  && this.isCurrentMonth(date)&&
        (!this.disabledStartDate || date >= this.disabledStartDate) &&
        (!this.disabledEndDate || date <= this.disabledEndDate)
      );
    },
    prevMonth() {
      this.currentDate.setMonth(this.currentDate.getMonth() - 1);
      this.currentDate = new Date(this.currentDate);
    },
    prevYear()
    {
      this.currentDate.setFullYear(this.currentDate.getFullYear() - 1);
      this.currentDate = new Date(this.currentDate);
    },
    nextYear()
    {
      this.currentDate.setFullYear(this.currentDate.getFullYear() + 1);
      this.currentDate = new Date(this.currentDate);
    },
   

    nextMonth() {
      this.currentDate.setMonth(this.currentDate.getMonth() + 1);
      this.currentDate = new Date(this.currentDate);
    },
    selectDate(date) {
      if (this.isDateEnabled(date)) {
        this.updateInputDate(date);
        this.$emit('updateselectedDate', date);
      }
    },
    isSelected(date) {
      if(!date || !this.selectDate )return false;
      return (
        this.selectedDate
        && date.getFullYear() === this.selectedDate.getFullYear()
        && date.getMonth() === this.selectedDate.getMonth()
        && date.getDate() === this.selectedDate.getDate()
      );
    },
  },
};
</script>
<style  scoped>
.calendar {
  display: flex;
  justify-content: center;
  width: 60%;
  margin: 0 auto;
  font-family: Arial, sans-serif;
  padding: 5px;
  background-color: #fff;
  border-radius: 10px;
  padding-left:55px ;
 
}

.contant {
  display: flex;
  height: 400px;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
}

.left {
  border-radius: 5px;
  padding: 10px 20px 20px 20px;
}
.inputdate{
  display: flex;
  justify-content: space-between;
  height: 40px;
  border-bottom: 1px solid #e4e4e4;
}
.icon-left{
  font-size: 18px;
}
td.selected{
  background-color:#409eff ;
  color: #fff;
  width: 24px;
  font-size: 12.5px;
  border: 4px solid #f2f6fc;
  height: 24px;
  border-radius: 50%; 
  display: flex;
  align-items: center;
  justify-content: center;
  transform: translateX(24%)  translateY(20%); 
}
td.not-current-month {
  color: #bfbfbf;
}
.left{
      border-right: 1px solid #e4e4e4;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 15px;
  color: #303133;
  margin-top: 20px;
}

.date-content-right{
  flex: 1;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  height: 80%;
  border-bottom: 1px solid #e4e4e4;
}

th {
  font-size: 13px;
  font-weight: normal;
  padding-bottom: 5px;
  color: #1a1a1b;

}

td {
  width: 14%;
  height: 35px;
  text-align: center;
  font-size: 14px;
}
thead{
  border-bottom: 1px solid #e4e4e4;
}




:deep(.el-input__inner)
{
height:30px;
width: 150px;
}

</style>