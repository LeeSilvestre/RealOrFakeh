<template>
  <main>
      <div class="top-container">
        <h1 class="bg-title">Book Inventory</h1>
        <div class="studentProfile">
              <span class="material-icons">inventory</span>
              <span class="text">Book Inventory</span>
        </div>
        <div class="buttons">
            <v-btn class="mb-2" color="#28a745" dark @click="printReport">Generate Report</v-btn>
        </div>
        <div class="buttons">
            <v-btn class="mb-2" color="primary" dark @click="manageCategories">Manage Categories</v-btn>
        </div>
        <span></span>
        <div class="buttons" style="margin-right: 10px;">
            <v-btn class="mb-2" color="#B71C1C" dark style="color: white;" @click="manageArchives">Archives</v-btn>
        </div>

      </div>  
      <hr>

      <div class="main-content">

          <div class="student-table">
              <StudentTable ref="student-table"/>
          </div>
      </div>

      
</main>

</template>

<script>
// import AddStudent from '../components/AddStudent.vue';
import StudentTable from '../components/StudentTable.vue';
import * as XLSX from 'xlsx';
import Swal from 'sweetalert2';
import axios from 'axios';

export default {
  data() {
    return {
      books: [],
    };
  },
components: {
  // AddStudent,
  StudentTable
},
methods:{
  async fetchBooks() {
      try {
        const response = await axios.get('/library/books'); 
        this.books = response.data;
      }catch{}
    },
    manageCategories() {
      this.$router.push('/categories');
    },
    manageArchives() {
      this.$router.push('/bookarchives');
    },
    async printReport() {
      await this.fetchBooks();

      if (!this.books.length) {
        Swal.fire({ 
          icon: 'warning',
          title: 'No Data',
          text: 'No books available to generate a report.',
        });
        return;
      }

      const workbook = XLSX.utils.book_new();
      const worksheetData = this.books.map(book => ({
        DDC: book.category.book_ddc,
        TITLE: book.book_title,
        AUTHOR: book.book_auth,
        CATEGORY: book.category.categ_name,
        'PUB. DATE': this.formatDate(book.pub_date),
        'C/O': book.copyright_owner,
        'NO. OF COPIES': book.book_qty,
      }));

      const worksheet = XLSX.utils.json_to_sheet(worksheetData);

      const columnWidths = [
        { wch: 22 }, // Column width for DDC
        { wch: 22 }, // Column width for TITLE
        { wch: 22 }, // Column width for AUTHOR
        { wch: 22 }, // Column width for CATEGORY
        { wch: 22 }, // Column width for PUB. DATE
        { wch: 22 }, // Column width for C/O
        { wch: 22 }, // Column width for NO. OF COPIES
      ];

      worksheet['!cols'] = columnWidths;

      worksheet['!rows'] = worksheetData.map(() => ({
        hpt: 16,
        hpx: 16
      }));

      for (let cell in worksheet) {
        if (cell[0] === '!') continue; // Skip metadata
        worksheet[cell].s = {
          alignment: {
            horizontal: 'center',
            vertical: 'center',
          }
        };
      }

      XLSX.utils.book_append_sheet(workbook, worksheet, 'Book Inventory');
      
      XLSX.writeFile(workbook, `Book_Inventory_Report_${new Date().toISOString().split('T')[0]}.xlsx`);
      
      Swal.fire({
        icon: 'success',
        title: 'Report Generated',
        text: 'The book inventory report has been generated successfully!',
      });
    },

    formatDate(date) {
      return new Date(date).toLocaleDateString();
    },
},
};
</script>

<style lang="scss" scoped>
.buttons {
  display: flex;
  align-items: center;
  margin-right: 20px;
}
.top-container{
            display: flex;
            margin: 0.5rem;
  
            .bg-title{
                z-index: -1;
                position: absolute;
                opacity: 10%;
                margin-top: -40px;
                font-size: 100px;
                font-family: Impact, fantasy;
            }
            .studentProfile{
                flex: 1;
        
                .material-icons{
                    position: relative;
                    color: var(--dark);
                    font-size: 2.5rem;
                }
                .text{
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

.main-content {
  display: flex;
  flex-direction: column;
  margin-right: 10px;

  .student-table{
      border-radius: 10px;
      background-color: #f0f0f0;
      padding: 0.5rem;
      height: 100%;
  }
}

</style>