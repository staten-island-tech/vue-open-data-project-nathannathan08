
<template>
  <div>
    <h1>Hate Crime Data in NYC - Motive</h1>
    <h3>2019-2024</h3>

    <label for="yearSearch">Filter by Year (2019-2024):</label>
    <input 
      id="yearSearch" 
      type="number" 
      v-model="searchYear" 
      @input="filterDataByYear" 
      min="2019" 
      max="2024" 
      placeholder="Enter Year" 
    />

    <canvas id="motiveChart"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, nextTick } from "vue";
import Chart from "chart.js/auto";


const searchYear = ref('');
const hate = ref([]);
const motiveChart = ref(null);


async function getData() {
  try {
    let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
    if (!res.ok) throw new Error("Failed to fetch data");
    let data = await res.json();
    hate.value = data;
  } catch (error) {
    console.error(error);
    alert("Failed to fetch data");
  }
}

function filterDataByYear() {
  nextTick(() => {
    renderChart();
  });
}

const filteredHateData = computed(() => {
  if (!searchYear.value) {
    return hate.value; 
  }
  
  const year = parseInt(searchYear.value);
  if (year >= 2019 && year <= 2024) {
    return hate.value.filter(item => {
      const incidentYear = new Date(item.incident_date).getFullYear();
      return incidentYear === year;
    });
  }
  return []; 
});


const MotivesWithCounts = computed(() => {
  const MotiveCounts = filteredHateData.value.reduce((acc, item) => {
    const category = item.bias_motive_description || "Unknown";
    acc[category] = (acc[category] || 0) + 1;
    return acc;
  }, {});

  const categories = Object.keys(MotiveCounts);
  const categorizedData = categories.reduce((acc, category) => {
    const count = MotiveCounts[category];
    if (count < 10) {
      acc.Other = (acc.Other || 0) + count;
    } else {
      acc[category] = count;
    }
    return acc;
  }, {});

  return Object.keys(categorizedData).map(category => ({
    bias_motive_description: category,
    count: categorizedData[category]
  }));
});


const renderChart = () => {
  const chartData = {
    labels: MotivesWithCounts.value.map(item => item.bias_motive_description),
    datasets: [
      {
        label: "Motive Counts",
        data: MotivesWithCounts.value.map(item => item.count),
        backgroundColor: ["#FF5733", "#33FF57", "#3357FF", "#F1C40F", "#9B59B6", "#ff005d", "#7c00ba", "#007328", "#ff3300", "#63c6ff", "#4f4f4f", "#ff4f4f", "#eeff00"], // Customize colors
        hoverOffset: 6
      }
    ]
  };


  if (motiveChart.value) {
    motiveChart.value.destroy();
  }

  motiveChart.value = new Chart(document.getElementById("motiveChart"), {
    type: "pie",
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
  color: rgb(133, 0, 11);
  font-size: 44px;
}
h3 {
  color: rgb(133, 0, 11);
}
input {
  margin: 10px;
  padding: 5px;
  font-size: 16px;
  width: 150px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>
