<template>
  <div>
    <h1>Hate Crime Data in NYC - Motive</h1>

    <ul>
      <li v-for="(item, index) in MotivesWithCounts" :key="index">
        <p>Motive: {{ item.bias_motive_description }}, Count: {{ item.count }}</p>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";

// Define your `hate` variable
const hate = ref([]);

// Fetch data from the API
async function getData() {
  let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
  let data = await res.json();
  hate.value = data;
}

// Computed property to calculate and group offenses by category with counts
const MotivesWithCounts = computed(() => {
  const MotiveCounts = hate.value.reduce((acc, item) => {
    const category = item.bias_motive_description || "Unknown"; 
    acc[category] = (acc[category] || 0) + 1;
    return acc;
  }, {});

  return Object.keys(MotiveCounts).map(category => ({
    category,
    count: MotiveCounts[category]
  }));
});

onMounted(() => {
  getData();
});
</script>

<style scoped>
ul {
  list-style-type: none;
    
}

li {
  border: 1px solid #ddd;
  margin: 10px 0;
  padding: 10px;
   
}

p {
  margin: 5px 0;
  color: rgb(168, 0, 22);
  font-size: 35px;
    
}
</style>
