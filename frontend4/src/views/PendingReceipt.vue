<template>
    <main>
      <div class="top-container">
        <h1 class="bg-title">With Fine</h1>
        <div class="Records">
          <span class="material-icons">pending</span>
          <span class="text">With Fine</span>
        </div>
      </div>  
      <hr>
  
      <div class="main-content">
        <div class="record-table">
          <RecordTable/>
        </div>
      </div>
    </main>
  </template>
  
  <script>
  import RecordTable from '../components/PendingReceiptTable.vue';
  
  export default {
    components: {
      RecordTable
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
  
    .Records {
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
  
    .record-table {
      border-radius: 10px;
      background-color: #f0f0f0;
      padding: 0.5rem;
      height: 100%;
    }
  }
  </style>