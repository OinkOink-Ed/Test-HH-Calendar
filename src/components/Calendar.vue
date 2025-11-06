<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const props = defineProps({
  initialDate: {
    type: String,
    default: null,
  },
  language: {
    type: String,
    default: 'ru',
  },
});

const emit = defineEmits(['date-selected']);

const currentDate = ref(new Date());
const selectedDate = ref(new Date());

const languages = {
  en: {
    months: [
      'January',
      'February',
      'March',
      'April',
      'May',
      'June',
      'July',
      'August',
      'September',
      'October',
      'November',
      'December',
    ],
    weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
  },
  ru: {
    months: [
      'Январь',
      'Февраль',
      'Март',
      'Апрель',
      'Май',
      'Июнь',
      'Июль',
      'Август',
      'Сентябрь',
      'Октябрь',
      'Ноябрь',
      'Декабрь',
    ],
    weekdays: ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'],
  },
};

const year = computed(() => currentDate.value.getFullYear());
const month = computed(() => currentDate.value.getMonth());

const formattedMonth = computed(() => {
  const monthName =
    languages[props.language]?.months[month.value] ||
    languages.en.months[month.value];
  return `${monthName} ${year.value}`;
});

const weekdays = computed(() => {
  return languages[props.language]?.weekdays || languages.en.weekdays;
});

const days = computed(() => {
  const daysArray = [];
  const firstDay = new Date(year.value, month.value, 1);
  const lastDay = new Date(year.value, month.value + 1, 0);

  // Определяем день недели первого дня месяца (0 - воскресенье, 1 - понедельник, и т.д.)
  const firstDayOfWeek = firstDay.getDay();

  // Создаем пустые ячейки для дней до первого дня месяца
  for (let i = 0; i < firstDayOfWeek; i++) {
    daysArray.push(null); // Пустая ячейка
  }

  // Добавляем дни текущего месяца
  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(year.value, month.value, i);
    daysArray.push({
      number: i,
      date: date,
      isCurrentMonth: true,
      isToday: isToday(date),
      isSelected: isSelected(date),
    });
  }

  // Добиваем массив до 42 ячеек (6 недель) пустыми значениями
  while (daysArray.length < 35) {
    daysArray.push(null);
  }

  return daysArray;
});

const initializeDate = () => {
  if (props.initialDate) {
    const [year, month, day] = props.initialDate.split('-').map(Number);
    currentDate.value = new Date(year, month - 1, day);
    selectedDate.value = new Date(year, month - 1, day);
  } else {
    const today = new Date();
    currentDate.value = today;
    selectedDate.value = today;
  }
};

const previousMonth = () => {
  currentDate.value = new Date(year.value, month.value - 1, 1);
};

const nextMonth = () => {
  currentDate.value = new Date(year.value, month.value + 1, 1);
};

const selectDate = (day) => {
  selectedDate.value = day.date;
  emit('date-selected', formatDate(day.date));
};

const isToday = (date) => {
  const today = new Date();
  return date.toDateString() === today.toDateString();
};

const isSelected = (date) => {
  if (!selectedDate.value) return false;
  return date.toDateString() === selectedDate.value.toDateString();
};

const formatDate = (date) => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

onMounted(() => {
  initializeDate();
});

watch(
  () => props.initialDate,
  () => {
    initializeDate();
  },
);
</script>

<template>
  <div class="calendar">
    <div class="calendar-header">
      <button @click="previousMonth">&lt;</button>
      <span>{{ formattedMonth }}</span>
      <button @click="nextMonth">&gt;</button>
    </div>

    <div class="weekdays">
      <div v-for="day in weekdays" :key="day" class="weekday">
        {{ day }}
      </div>
    </div>

    <div class="days-grid">
      <div
        v-for="(day, index) in days"
        :key="day ? day.date.toString() : `empty-${index}`"
        :class="[
          'day',
          {
            selected: day && day.isSelected,
            empty: !day,
          },
        ]"
        @click="day && selectDate(day)"
      >
        {{ day ? day.number : '' }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar {
  height: 300px;
  font-family: Arial, sans-serif;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

button {
  background: none;
  border: 1px solid #ccc;
  padding: 5px 10px;
  cursor: pointer;
  border-radius: 3px;
  color: inherit;
}

button:hover {
  background-color: #f0f0f0;
}

.weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  margin-bottom: 5px;
}

.weekday {
  font-weight: bold;
  padding: 5px;
}

.days-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}

.day {
  padding: 5px;
  cursor: pointer;
  border: 1px solid transparent;
}

.day.empty {
  cursor: default;
  background-color: transparent;
}

.day.selected {
  background-color: #007bff;
  color: white;
}

.day:not(.empty):hover {
  background-color: #f0f0f0;
}

.day.selected:hover {
  background-color: #0056b3;
}
</style>
