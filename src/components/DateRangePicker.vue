<template>
    <div>
      <label for="startDate">Start Date</label>
      <input type="date" v-model="startDate" id="startDate" />
  
      <label for="endDate">End Date</label>
      <input type="date" v-model="endDate" id="endDate" />
  
      <button @click="applyFilter">Apply Filter</button>
    </div>
  </template>
  
  <script setup>
  import { ref, defineEmits } from 'vue';
  
  // Reactive variables for start and end dates
  const startDate = ref('');
  const endDate = ref('');
  
  // Emit the dates when the "Apply Filter" button is clicked
  const emit = defineEmits();
  
  // Emit the selected date range to the parent component
  const applyFilter = () => {
    const dates = { startDate: startDate.value, endDate: endDate.value };
    emit('filter', dates); // Emit the event to notify the parent component
  };
  </script>
  
  <style scoped>
  input[type="date"] {
    margin: 10px;
  }
  button {
    margin-top: 10px;
  }
  </style>
  