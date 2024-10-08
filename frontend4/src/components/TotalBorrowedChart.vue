<template>
  <main>
    <div class="top-container">
      <!-- Existing content here -->
    </div>
    <hr>
    <div class="main-content">
      <!-- Date filter -->
      <!-- Bar chart component -->
      <div class="chartdes">
        <Bar
          ref="barChart"
          :chart-id="'bar-chart'"
          :dataset-id="'bar-dataset'"
          :data="chartData"
          :options="chartOptions"
        />
        <!-- Popup or Modal for showing details -->
        <transition name="fade">
          <div v-if="showDetails" class="popup">
            <div class="popup-content">
              <p><strong>Most Borrowed Book:</strong></p>
              <h2>{{ selectedSubject.title }}</h2>
              <p>Author: {{ selectedSubject.author }}</p>
              <button @click="closePopup">Close</button>
            </div>
          </div>
        </transition>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue';
import { Bar } from 'vue-chartjs';
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
} from 'chart.js';
import axios from 'axios';

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);

// Reactive variables
const categories = ref([]);
const books = ref([]);
const borrowStatuses = ref([]);
const months = [
  'January', 'February', 'March', 'April', 'May', 'June',
  'July', 'August', 'September', 'October', 'November', 'December'
];

const chartData = computed(() => {
  // Create a map to count borrowed books per category
  const categoryBorrowCounts = new Map(categories.value.map(c => [c.categ_name, 0]));

  borrowStatuses.value.forEach(borrowStatus => {
    const book = books.value.find(b => b.book_title === borrowStatus.book_title);
    if (book) {
      const category = book.categ_name;
      if (categoryBorrowCounts.has(category)) {
        categoryBorrowCounts.set(category, categoryBorrowCounts.get(category) + 1);
      }
    }
  });

  const labels = Array.from(categoryBorrowCounts.keys());
  const data = Array.from(categoryBorrowCounts.values());

  // Determine the most borrowed book
  let favoriteBook = '';
  let maxBorrowed = 0;

  labels.forEach(label => {
    const categoryBooks = books.value.filter(b => b.categ_name === label);
    const borrowedBooks = borrowStatuses.value.filter(borrowStatus => {
      const book = categoryBooks.find(bk => bk.book_title === borrowStatus.book_title);
      return book;
    });

    borrowedBooks.forEach(borrowStatus => {
      const bookCount = borrowedBooks.filter(borrowedBook => borrowedBook.book_title === borrowStatus.book_title).length;
      if (bookCount > maxBorrowed) {
        maxBorrowed = bookCount;
        favoriteBook = borrowStatus.book_title;
        selectedSubject.value = {
          title: borrowStatuses.value.find(bs => bs.book_title === favoriteBook)?.book_title || 'Unknown Title',
          author: books.value.find(bk => bk.book_title === favoriteBook)?.book_auth || 'Unknown Author',
          favoriteBook: favoriteBook,
        };
      }
    });
  });

  return {
    labels,
    datasets: [
      {
        label: 'TOTAL BOOK USE',
        backgroundColor: '#2f3f64',
        data,
      },
    ],
  };
});

const chartOptions = ref({
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    x: {
      stacked: true,
      title: {
        display: true,
        text: 'Categories',
      },
    },
    y: {
      stacked: true,
      beginAtZero: true,
      title: {
        display: true,
        text: 'TOTAL BORROWED BOOKS',
      },
    },
  },
  plugins: {
    legend: {
      display: false,
    },
  },
  onClick: handleChartClick,
});

const showDetails = ref(false);
const selectedSubject = ref({
  title: '',
  author: '',
  favoriteBook: 'Favorite Book',
});

const barChart = ref(null);

async function fetchData() {
  try {
    // Fetch all borrow records first
    const borrowStatusesResponse = await axios.get('/library/borrowstatus');
    borrowStatuses.value = borrowStatusesResponse.data;

    // Fetch all book details
    const booksResponse = await axios.get('/library/books');
    books.value = booksResponse.data;

    // Fetch all categories
    const categoriesResponse = await axios.get('/library/category');
    categories.value = categoriesResponse.data;

    console.log('Borrow Statuses:', borrowStatuses.value);
    console.log('Books:', books.value);
    console.log('Categories:', categories.value);

    await nextTick(); // Ensure DOM is updated before chart update
    updateChartData();
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

function updateChartData() {
  console.log('Updating chart data...');
  // Force chart to update
  barChart.value?.chart.update('none');
}

function handleChartClick(event, elements) {
  if (elements.length) {
    const elementIndex = elements[0].index;
    const label = chartData.value.labels[elementIndex];

    // Find the corresponding category
    const categoryData = categories.value.find(c => c.categ_name === label);
    if (categoryData) {
      // Find the most borrowed book in the selected category
      const borrowedBooksInCategory = borrowStatuses.value.filter(borrowStatus => {
        const book = books.value.find(b => b.book_title === borrowStatus.book_title);
        return book && book.categ_name === categoryData.categ_name;
      });

      if (borrowedBooksInCategory.length > 0) {
        // Find the most frequently borrowed book in the filtered list
        const bookCounts = borrowedBooksInCategory.reduce((counts, borrowStatus) => {
          counts[borrowStatus.book_title] = (counts[borrowStatus.book_title] || 0) + 1;
          return counts;
        }, {});

        const favoriteBookTitle = Object.keys(bookCounts).reduce((a, b) => bookCounts[a] > bookCounts[b] ? a : b);
        const favoriteBook = books.value.find(b => b.book_title === favoriteBookTitle);

        selectedSubject.value = {
          title: favoriteBook ? favoriteBook.book_title : 'Unknown Title',
          author: favoriteBook ? favoriteBook.book_auth : 'Unknown Author',
          favoriteBook: favoriteBookTitle
        };
      } else {
        selectedSubject.value = {
          title: 'Unknown Title',
          author: 'Unknown Author',
          favoriteBook: ''
        };
      }

      showDetails.value = true;
    }
  }
}

function closePopup() {
  showDetails.value = false;
}

onMounted(fetchData);
</script>
  
<style scoped>
/* Container for the bar chart */
.chartdes {
  width: 99%;
  height: 300px;
  margin: auto;
  position: relative;
  background: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  padding: 10px;
}

/* Styling for the popup/modal */
.popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  z-index: 1000;
  max-width: 80%;
  width: 300px;
  animation: fadeIn 0.3s ease-out;
}

.popup-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  background-color: #2f3f64;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #1e2a47;
}

/* Fade in/out animation for the popup */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}

/* Alerts container styles */
.alerts-container {
  margin-top: 2rem;
  padding: 1rem;
}

.alert-item {
  margin-bottom: 1rem;
  display: flex;
  justify-content: center;
}

.alert-card {
  border-left: 4px solid #2f3f64;
  text-align: center; /* Center-align the text */
  padding: 1rem; /* Optional: Add padding for better spacing */
}

.alert-title {
  font-weight: 700;
  margin-left: 0.5rem;
}

.material-icons {
  color: #2f3f64;
}


/* General body styling */
body {
  background-color: #f0f2f5;
  color: #333;
  font-family: 'Arial', sans-serif;
}
</style>
  