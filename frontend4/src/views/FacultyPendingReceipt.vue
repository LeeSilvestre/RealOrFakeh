<template>
    <main>
      <div class="top-container">
        <h1 class="bg-title">Faculty With Fine</h1>
        <div class="pendingBooks">
          <span class="material-icons">pending</span>
          <span class="text">Faculty With Fine</span>
        </div>
      </div>  
      <hr>
  
      <div class="main-content">
        <div class="pending-table">
          <PendingTable/>
        </div>
      </div>
    </main>
  </template>
  
  <script>
  import PendingTable from '../components/FacultyReceiptTable.vue';
  
  export default {
    components: {
      PendingTable
    }
  };
  </script>
  
  <style lang="scss" scoped>
  .top-container {
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

  
    .pendingBooks {
      flex: 1;
  
      .material-icons {
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
  
    .pending-table {
      border-radius: 10px;
      background-color: #f0f0f0;
      padding: 0.5rem;
      height: 100%;
    }
  }
  </style>
  