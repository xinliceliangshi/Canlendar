<template>
  <div id="app">
    <Calendar
      :selected-date="selectedDate"
      @updateselectedDate="selectDate"
      />
  </div>
</template>

<script>
import Calendar from '../src/views/HomeView.vue';

export default {
  name: 'App',
  components: {
    Calendar,
  },
  data() {
    return {
     selectedDate: new Date(),
    };
  },
  methods: {
    selectDate(date) {
      this.selectedDate = date;
    },
  },
};
</script>