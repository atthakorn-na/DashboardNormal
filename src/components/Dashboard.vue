<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <v-card class="mx-auto">
        <h1>Data Table User </h1>
      <v-table fixed-header>
        <thead>
          <tr>
            <th scope="col" class="text-center">ID</th>
            <th scope="col" class="text-center">Name</th>
            <th scope="col" class="text-center">City</th>
            <th scope="col" class="text-center">Average rate</th>
            <th scope="col" class="text-center">Vote</th>
          </tr>
        </thead>
    <tbody>
      <tr class="da"
      v-for="item in filteredList" :key="item.id"
      >
      <td >{{ item.id }}</td>
        <td >{{ item.name }}</td>
        <td >{{ item.city }}</td>
        <td >{{ item.user_rating.average_rating }}</td>
        <td >{{ item.user_rating.votes }}</td>
      </tr>
    </tbody>
  </v-table>     
      <v-row class="pagination">
        <v-col cols="12" class="text-center">
          <v-pagination v-model="currentPage" :length="totalPages" :total-visible="6" @input="changePage"></v-pagination>
        </v-col>
      </v-row>
  </v-card>
  <!-- <bar :data="chartData" :options="chartOptions" ></bar> -->
  
        <v-card class="chartCard">
        <h1>Chart User Rating </h1>
        <!--refresh page  chart not show-->

        <bar v-if="loaded" :data="chartData" :options="chartOptions" :key="chartKey" ></bar>  
      </v-card>
    </v-responsive>
  </v-container>
</template>

<script >
import axios from 'axios';
import { Bar } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)
export default {
  components: {
    Bar
  },
  data() {
    return {
      list: [],
      currentPage: 1,
      totalData: 0,
      itemsPerPage: 10,
      chartKey: 0,
      loaded: false,
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Average rate',
            backgroundColor: 'rgba(75, 192, 192, 0.2)',
            borderColor: 'rgba(75, 192, 192, 1)',
            data: []
          }
        ]
      },
      chartOptions: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            max: 5
          }
        }
      }
    };
  },
  created() {
    this.fetchData(this.currentPage)
    // this.populateChartData();

  },

  computed: {
  filteredList() {
    const start = (this.currentPage - 1) * this.itemsPerPage;
    const end = start + this.itemsPerPage;
    return this.list.slice(start, end);
  },
  totalPages() {
    return Math.ceil(this.totalData / this.itemsPerPage);
  }
},
  methods: {
    async fetchData(page) {
      try {
        const response = await axios.get(`https://jsonmock.hackerrank.com/api/food_outlets?page=${page}`);
        this.list = response.data.data;
        this.totalData = response.data.total; // เพิ่มการอัพเดตค่า totalPages
        this.populateChartData();
        console.log(page)
        this.loaded = true
      } catch (error) {
        console.error(error);
      }
    },
    populateChartData() {
      this.chartData.labels = this.filteredList.map((item) => item.name);
      this.chartData.datasets[0].data = this.filteredList.map((item) => item.user_rating.average_rating);
      this.chartKey++; // idk why
      console.log(this.chartData.labels)
      console.log(this.chartData.datasets[0].data)
    },
    changePage(page) {
      this.currentPage = page;
      this.loaded = false; // ตั้งค่า loaded เป็น false เพื่อแสดงว่ากำลังโหลดข้อมูลใหม่
      this.fetchData().then(() => {
        this.populateChartData();
        this.loaded = true; // ตั้งค่า loaded เป็น true เมื่อโหลดข้อมูลเสร็จสิ้น
      });
    }
  }
};
</script>

<style scoped>
.da:hover{
  background-color: #7FC696;
}
.pagination {
  margin-top: 20px;
}
.pagination .v-pagination {
  display: inline-block;
  cursor: pointer;
}

.pagination .v-pagination .v-pagination__item {
  margin-right: 4px;
}

.pagination .v-pagination .v-pagination__item--active {
  background-color: #7FC696;
  color: white;
}

.pagination .v-pagination .v-pagination__item--disabled {
  cursor: not-allowed;
}
</style>
