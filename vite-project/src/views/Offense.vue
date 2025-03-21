
<template>
  <div>
    <h1>Hate Crime Data in NYC - Offense</h1>

   <h2>Select an Offensive Category</h2>
    <select v-model="selectedCategory" @change="updateChart">
      <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
      <option value="All">All Categories</option>
    </select>

    <canvas id="offenseChart"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, nextTick } from "vue";
import Chart from "chart.js/auto"; 

const hate = ref([]);
const offenseChart = ref(null);
const selectedCategory = ref("All");


const categories = [
  "Religion/Religious Practice",
  "Sexual Orientation",
  "Race/Color",
  "Gender",
  "Ethnicity/National Origin/Ancestry",
  "Unclassified"
];

async function getData() {
  try {
    let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
    if (!res.ok) throw new Error("Failed to fetch data");
    let data = await res.json();
    hate.value = data;
    console.log(hate.value);  
  } catch (error) {
    console.error(error);
    alert("Failed to fetch data");
  }
}


const filteredData = computed(() => {
  if (selectedCategory.value === "All") {
    return hate.value;
  }
 
  return hate.value.filter(item => item.offense_category === selectedCategory.value);
});

const OffenseWithCounts = computed(() => {
  const OffenseCounts = filteredData.value.reduce((acc, item) => {
    const category = item.offense_description || "Unknown";
    acc[category] = (acc[category] || 0) + 1;
    return acc;
  }, {});

  const categories = Object.keys(OffenseCounts);
  const categorizedData = categories.reduce((acc, category) => {
    const count = OffenseCounts[category];
    if (count < 10) {
      acc.Other = (acc.Other || 0) + count;
    } else {
      acc[category] = count;
    }
    return acc;
  }, {});

  return Object.keys(categorizedData).map(category => ({
    offense_description: category,
    count: categorizedData[category],
  }));
});

const renderChart = () => {
  const chartData = {
    labels: OffenseWithCounts.value.map(item => item.offense_description),
    datasets: [
      {
        label: "Offense Counts",
        data: OffenseWithCounts.value.map(item => item.count),
        backgroundColor: ["#8a0017"], 
        hoverOffset: 4,
      },
    ],
  };

  if (offenseChart.value) {
    offenseChart.value.destroy(); 
  }

  offenseChart.value = new Chart(document.getElementById("offenseChart"), {
    type: "bar",
    data: chartData,
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: "top",
        },
        tooltip: {
          callbacks: {
            label: function (tooltipItem) {
              return `${tooltipItem.label}: ${tooltipItem.raw} incidents`;
            },
          },
        },
      },
    },
  });
};

const updateChart = () => {
  nextTick(() => renderChart());
};

onMounted(async () => {
  await getData();
  nextTick(() => renderChart());
});
</script>

<style scoped>
canvas {
  max-width: 900px; 
  width: 100%;     
  height: 300px;  
  margin: 0 auto;  
}
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
  font-size: 44px;
}
h2 {
  color:rgb(133, 0, 11)
}
h3 {
  color:rgb(133, 0, 11)
}
</style>
