<script setup>
import Calendar from './components/Calendar.vue';

const handleDateSelect = (date) => {
  console.log('Выбрана дата:', date);
};
</script>

<template>
  <Calendar language="en" @date-selected="handleDateSelect" />
</template>

<style scoped></style>
