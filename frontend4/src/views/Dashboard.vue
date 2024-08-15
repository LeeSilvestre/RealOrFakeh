<template>
  <main>
    <div class="top-container">
      <h1 class="bg-title">Dashboard</h1>
      <div class="total-student">
        <span class="material-icons">dashboard</span>
        <span class="text">Dashboard</span>
      </div>
    </div>
    <hr>

    <!-- Combined container with cards and charts -->
    <div class="container">
      <div class="card">
        <div class="total-student">
          <h1 class="info-text">
            <span class="material-icons">feed</span>
            BOOK TITLES
          </h1>
          <br>
          <h2>Total: {{ totalBooks }}</h2>
        </div>
      </div>
      <div class="card">
        <div class="total-student">
          <h1 class="info-text">
            <span class="material-icons">pending</span>
            STUDENT BORROWED BOOKS
          </h1>
          <h2>Total: {{ totalPendingBooks }}</h2>
        </div>
      </div>
      <div class="card">
        <div class="total-student">
          <h1 class="info-text">
            <span class="material-icons">history_edu</span>
            FACULTY BORROWED BOOKS
          </h1>
          <h2>Total: {{ totalFacultyPendingBooks }}</h2>
        </div>
      </div>
    </div>

    <div class="charts-row">
      <div class="card big-chart-card">
        <h1 class="info-text">
          <span class="material-icons">bar_chart</span>
          MONTHLY LIBRARY LOGS
        </h1>
        <canvas ref="summaryChartCanvas"></canvas>
      </div>
      <div class="card big-chart-card pie-chart-card">
        <h1 class="info-text">
          <span class="material-icons">pie_chart</span>
          BOOKS PER CATEGORY
        </h1>
        <canvas ref="pieChartCanvas"></canvas>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import {  
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  BarController,
  CategoryScale,
  LinearScale,
  ArcElement,
  PieController
} from 'chart.js';

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  BarElement,
  BarController,
  CategoryScale,
  LinearScale,
  ArcElement,
  PieController
);

const totalBooks = ref(0);
const totalPendingBooks = ref(0);
const totalFacultyPendingBooks = ref(0);
const timeData = ref([]);
const bookCategories = ref([]);
const summaryChartCanvas = ref(null);
const pieChartCanvas = ref(null);
const bookCategoryCounts = ref({});

const fetchData = async () => {
  try {
    const booksResponse = await axios.get('/library/books');
    const categoriesResponse = await axios.get('/library/category');
    const borrowStatusResponse = await axios.get('/library/borrowstatus');
    const facultyBorrowsResponse = await axios.get('/library/faculty/borrows');
    const logsResponse = await axios.get('/library/logs');

    console.log('Books:', booksResponse.data);
    console.log('Categories:', categoriesResponse.data);
    console.log('Borrow Status:', borrowStatusResponse.data);
    console.log('Faculty Borrows:', facultyBorrowsResponse.data);
    console.log('Logs:', logsResponse.data);

    totalBooks.value = booksResponse.data.length;
    totalPendingBooks.value = borrowStatusResponse.data.filter(
      item => [0, 1, 5, 6, 7].includes(item.borrow_status)
    ).length;
    totalFacultyPendingBooks.value = facultyBorrowsResponse.data.filter(
      item => [0, 1, 5, 6, 7].includes(item.borrow_status)
    ).length;
    bookCategories.value = categoriesResponse.data;

    processBooksData(booksResponse.data);
    calculateMonthlyLogs(logsResponse.data);
    renderSummaryChart();
    renderPieChart();
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

const calculateMonthlyLogs = (logs) => {

  console.log('Logs:', logs);
  const monthlyCounts = Array(12).fill(0);

  logs.forEach(log => {
    const month = new Date(log.login_date).getMonth();
    if (month >= 0 && month < 12) {
      monthlyCounts[month]++;
    }
  });

  console.log('Monthly Counts:', monthlyCounts);

  timeData.value = monthlyCounts.map((count, index) => ({
    label: new Date(0, index).toLocaleString('en-US', { month: 'short' }),
    count
  }));
};

const processBooksData = (books) => {
  const categoryCounts = {};

  books.forEach(book => {
    const category = book.categ_name;
    if (category) {
      categoryCounts[category] = (categoryCounts[category] || 0) + 1;
    }
  });

  // Update the reactive reference
  bookCategoryCounts.value = categoryCounts;
};

const renderSummaryChart = () => {
  if (summaryChartCanvas.value) {
    new ChartJS(summaryChartCanvas.value.getContext('2d'), {
      type: 'bar',
      data: {
        labels: timeData.value.map(item => item.label),
        datasets: [{
          label: 'Count',
          data: timeData.value.map(item => item.count),
          backgroundColor: 'rgba(75, 192, 192, 0.2)',
          borderColor: 'rgba(75, 192, 192, 1)',
          borderWidth: 1
        }]
      },
      options: {
        scales: { 
          y: { 
            beginAtZero: true,
            ticks: {
              stepSize: 1, 
              callback: function(value) { 
                if (Number.isInteger(value)) {
                  return value;
                }
              }
            }
          } 
        },
        responsive: true,
        maintainAspectRatio: false
      }
    });
  }
};

const renderPieChart = () => {
  if (pieChartCanvas.value) {
    new ChartJS(pieChartCanvas.value.getContext('2d'), {
      type: 'pie',
      data: {
        labels: Object.keys(bookCategoryCounts.value),
        datasets: [{
          label: 'Books per Category',
          data: Object.values(bookCategoryCounts.value),
          backgroundColor: [
            'rgba(75, 192, 192, 0.2)', 'rgba(153, 102, 255, 0.2)', 'rgba(255, 159, 64, 0.2)',
            'rgba(255, 205, 86, 0.2)', 'rgba(54, 162, 235, 0.2)', 'rgba(255, 99, 132, 0.2)',
            'rgba(201, 203, 207, 0.2)'
          ],
          borderColor: [
            'rgba(75, 192, 192, 1)', 'rgba(153, 102, 255, 1)', 'rgba(255, 159, 64, 1)',
            'rgba(255, 205, 86, 1)', 'rgba(54, 162, 235, 1)', 'rgba(255, 99, 132, 1)',
            'rgba(201, 203, 207, 1)'
          ],
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    });
  }
};

onMounted(fetchData);
</script>

<style lang="scss" scoped>
.top-container {
  display: flex;
  margin: 0.5rem;

  .bg-title {
    z-index: -1;
    position: absolute;
    opacity: 10%;
    margin-top: -40px;
    font-size: 100px;
    font-family: Impact, fantasy;
  }

  .total-student {
    flex: 1;

    .material-icons {
      color: var(--dark);
      font-size: 2.5rem;
    }

    .text {
      position: relative;
      color: var(--dark);
      font-size: 2.5rem;
      font-weight: 900;
      bottom: 0.5rem;
      text-shadow: 0 1px 1px;
      margin-left: 1rem;
    }
  }
}

.container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: flex-start;
  width: 100%;
  margin-bottom: 1.5rem;
}

.card {
  flex: 1;
  margin: 0.5rem;
  padding: 1.5rem;
  background-color: #f0f0f0;
  color: var(--dark);
  border-radius: 5px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;

  &:hover {
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
  }

  .total-student {
    width: 100%;

    .info-text {
      display: flex;
      align-items: center;
      justify-content: center;
      text-shadow: 0 0 1px;
      font-size: 25px;
      margin-bottom: 1rem;
      text-align: center;

      .material-icons {
        font-size: 38px;
        margin-right: 0.5rem;
      }
    }

    h2 {
      margin-bottom: 1rem;
      font-weight: 900;
      text-align: center;
    }
  }
}

.charts-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  width: 100%;
  margin-bottom: 1rem;

  .big-chart-card, .pie-chart-card {
    flex: 1;
    margin: 0.5rem;
    height: 400px;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: #f0f0f0;
    padding: 1rem;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transition: box-shadow 0.3s ease;

    canvas {
      width: 85% !important;
      height: auto !important;
    }

    &:hover {
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
    }
  }
}

h1.info-text {
  display: flex;
  align-items: center;
  font-size: 20px;
  margin-bottom: 1rem;
  text-align: center;

  .material-icons {
    font-size: 36px;
    margin-right: 0.5rem;
  }
}
</style>
