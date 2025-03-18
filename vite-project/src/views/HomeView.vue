<template>
  <div>
    <h1>Hate Crime Data in NYC</h1>
    <ul>
      <li v-for="(item, index) in offenseCategoriesWithCounts" :key="index">
        <p>Offensive Category: {{ item.category }}, Count: {{ item.count }}</p>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";

const hate = ref([]);

async function getData() {
  let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
  let data = await res.json();
  hate.value = data;
}


const offenseCategoriesWithCounts = computed(() => {
  const offenseCounts = hate.value.reduce((acc, item) => {
    const category = item.offense_category || "Unknown"; 
    acc[category] = (acc[category] || 0) + 1;
    return acc;
  }, {});

  return Object.keys(offenseCounts).map(category => ({
    category,
    count: offenseCounts[category]
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

h1 {
  color:rgb(133, 0, 11);
  font-size: 50px;
}
</style>
