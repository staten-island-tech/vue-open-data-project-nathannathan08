<template>
  <div>
    <h1>Hate Crime Data in NYC</h1>

    <!-- Pie Chart Section -->
    <div>
      <canvas id="hateCrimePieChart"></canvas>
    </div>

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
import { Chart } from "chart.js";
 // Import Chart.js

const hate = ref([]);

async function getData() {
  let res = await fetch("https://data.cityofnewyork.us/resource/bqiq-cu78.json");
  let data = await res.json();
  hate.value = data;
  createPieChart(); // Create the pie chart after data is fetched
}

// Function to generate Pie Chart
function createPieChart() {
  const offenseCounts = hate.value.reduce((acc, item) => {
    const category = item.offense_category || "Unknown"; // Default to "Unknown" if category is missing
    acc[category] = (acc[category] || 0) + 1;
    return acc;
  }, {});

  const labels = Object.keys(offenseCounts);
  const data = Object.values(offenseCounts);

  new Chart(document.getElementById("hateCrimePieChart"), {
    type: "pie",
    data: {
      labels: labels,
      datasets: [{
        data: data,
        backgroundColor: ["#FF6384", "#36A2EB", "#FFCE56", "#4BC0C0", "#F7464A"], // Add more colors as needed
        hoverOffset: 4
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: {
          position: 'top',
        },
        tooltip: {
          callbacks: {
            label: function(tooltipItem) {
              return tooltipItem.label + ": " + tooltipItem.raw;
            }
          }
        }
      }
    }
  });
}

onMounted(() => {
  getData(); 
});
</script>

<style scoped>
#hateCrimePieChart {
  width: 400px;
  height: 400px;
  margin: 20px auto;
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
}

r {
  color: rgb(112, 0, 28);
}
</style>