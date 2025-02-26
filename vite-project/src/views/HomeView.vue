<template>
  <div>
    <h1>Hate Crime Data in NYC</h1>
    <ul>
      <li v-for="(item, index) in hate" :key="index">

        <r>Offense: {{ item.offense_description }}</r>
        <p>Motive: {{ item.bias_motive_description }}</p>
        <p>Offensive Category: {{ item.offense_category }}</p>

      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const hate = ref([]);


async function getData() {
  let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
  let data = await res.json();
  hate.value = data; 
}

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
  color:rgb(168, 0, 22)
}

r {
  color:rgb(112, 0, 28)
}
</style>
