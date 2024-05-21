<template>
    <div class="contain">
        <!-- Dashboard card for Absent -->
        <div class="card-absent">
            <div class="card-content-absent">
                <h2>Absent</h2>
                <p>{{ absent }}</p>
            </div>
        </div>

        <!-- Dashboard card for Attended -->
        <div class="card-absent">
            <div class="card-content-absent">
                <h2>Attended</h2>
                <p>{{ attended }}</p>
            </div>
        </div>
    </div>

    <div class="other-contain">
        <div class="card">
            <div class="card-content">
                <select v-model="selectedYear" class="select">
                    <option disabled selected value="">Year</option>
                    <option v-for="year in years" :key="year">{{ year }}</option>
                </select>
                <select v-model="selectedMonth" class="select">
                    <option disabled selected value="">Month</option>
                    <option v-for="(month, index) in months" :key="month" :value="index + 1">{{ month }}</option>
                </select>
                <button @click="fetchMonthlyAbsentData" class="button">Generate</button>
            </div>
            <div class="card-chart">
                <canvas id="barChart" width="400" height="300"></canvas>
            </div>
        </div>
    </div>

    <div class="new-comer-container">
        <h2 style="position: sticky;">New Commers</h2>
        <div class="new-comer-content">
            <div class="new-comer-elements">
                <div v-for="newComer in newCommers" :key="newComer.id" class="new-comer-card">
                    <p>{{ newComer.nama }} </p>
                    <p>{{ formatDate(newComer.tanggal) }}</p>
                </div>
            </div>
        </div>
    </div>

    <div id="app">
        <div class="new-comer-container">
            <h2 style="position: sticky;">Absent Students</h2>
            <div class="new-comer-content">
                <div class="new-comer-elements">
                    <div v-for="absentStudent in absentStudents" :key="absentStudent.id" class="new-comer-card">
                        <p>{{ absentStudent.nama }}</p>
                        <p>{{ formatDate(absentStudent.waktu_absen) }}</p>
                    </div>
                </div>
            </div>
        </div>
        <div id="app">
            <div class="new-comer-container">
                <h2 style="position: sticky;">Birthdays</h2>
                <div class="new-comer-content">
                    <div class="new-comer-elements">
                        <div v-for="birthday in studentBirthdays" :key="birthday.id" class="new-comer-card">
                            <p>{{ birthday.nama }}</p>
                            <p>{{ formatDate(birthday.tanggal) }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>


</template>

<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
    data() {
        return {
            selectedYear: '',
            selectedMonth: '',
            years: ['2022', '2023', '2024'], // Add years as needed
            months: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
            chart: null,
            absent: 0,
            attended: 0,
            chartData: {
                type: 'bar',
                labels: [],
                datasets: [{
                    label: 'Jumlah',
                    data: [],
                    // backgroundColor: 'rgba(54, 162, 235, 0.5)',
                    // borderColor: 'rgba(54, 162, 235, 1)',
                    // borderWidth: 1
                }]
            }, // Variable to store the chart instance
            newCommers: [],
            absentStudents: [],
            studentBirthdays: [],
        };
    },
    mounted() {
        // Render an empty chart on component mount
        this.renderChart([], []);
        this.fetchTodayAttendance();
        this.fetchNewCommers();
        this.fetchNewStudents();
        this.fetchBirthdays();

        // Panggil metode setiap 5 menit
        this.interval = setInterval(() => {
            this.fetchMonthlyAbsentData();
            this.fetchTodayAttendance();
            this.fetchNewCommers();
        }, 300000); // 300000 milidetik = 5 menit
    },
    unmounted() {
        // Hapus interval saat komponen dihancurkan untuk mencegah kebocoran memori
        clearInterval(this.interval);
    },

    methods: {
        fetchMonthlyAbsentData() {
            const year = this.selectedYear;
            const monthIndex = this.selectedMonth - 1;
            const month = (monthIndex < 9 ? '0' : '') + (monthIndex + 1).toString();
            console.log(month)
            console.log(year)

            axios.post('/absen/monthly-absent', { year, month })
                .then(response => {
                    console.log('Data fetched:', response.data);

                    // Extract jumlah and minggu lists from the response data
                    const jumlah = response.data.jumlah;
                    const minggu = response.data.minggu;


                    // Use jumlah and minggu as needed
                    console.log('Jumlah:', jumlah);
                    console.log('Minggu:', minggu);

                    this.renderChart(jumlah, minggu);
                })
                .catch(error => {
                    console.error('Error fetching monthly absent data:', error);
                });
        },
        renderChart(jumlah, minggu) {
            const formattedMinggu = minggu.map(dateString => {
                const date = new Date(dateString);
                const day = String(date.getDate()).padStart(2, '0');
                const month = String(date.getMonth() + 1).padStart(2, '0');
                const year = String(date.getFullYear()).slice(-2);
                return `${day}-${month}-${year}`;
            });
            // Update chart data
            this.chartData.labels = formattedMinggu;
            this.chartData.datasets[0].data = jumlah;

            // Destroy existing chart if it exists
            if (this.chart) {
                this.chart.destroy();
            }

            // Create a new chart instance
            this.chart = new Chart(document.getElementById('barChart'), {
                type: 'bar',
                data: this.chartData, // Use updated chart data
                options: {
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });
        },
        fetchTodayAttendance() {
            axios.post('absen/today-attendance')
                .then(response => {
                    this.absent = response.data.absent;
                    this.attended = response.data.attended;
                })
                .catch(error => {
                    console.error('Error fetching today attendance:', error);
                });
        },
        fetchNewCommers() {
            axios.post('absen/new-commers')
                .then(response => {
                    this.newCommers = response.data;
                    console.log(this.newCommers);
                })
                .catch(error => {
                    console.error('Error fetching new commers:', error);
                });
        },
        fetchNewStudents() {
            axios.post('absen/long-absent').then(
                response => {
                    this.absentStudents = response.data;
                    console.log(this.absentStudents)
                })
                .catch(
                    error => {
                        console.error('Error fetching absents', error);
                    }
                )
        },
        formatDate(dateString) {
            const date = new Date(dateString);
            const day = date.getDate().toString().padStart(2, '0');
            const month = (date.getMonth() + 1).toString().padStart(2, '0');
            const year = date.getFullYear();
            return `${day}/${month}/${year}`;
        },
        fetchBirthdays() {
            axios.post('absen/weekly-birthday').then(
                response => {
                    this.studentBirthdays = response.data;
                    console.log(this.studentBirthdays)
                })
                .catch(
                    error => {
                        console.error('Error fetching birthdays', error);
                    }
                )
        }
    },
};



</script>
<style>
.contain {
    /* display: flex; */
    /* flex-wrap: wrap; Allow cards to wrap to the next line */
    gap: 6px;
    /* Add some space between cards */
}

.other-contain {
    margin-left: 20px
}


/* new commer  */
.new-comer-container {
    margin-left: 20px;
    max-height: 400px;
    /* overflow-y: auto; */
    /* max-height: 200px; */
    border: 1px solid #ccc;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 20px;
}

.new-comer-content {
    overflow-y: auto;
    max-height: 300px;
    /* border: 1px solid #ccc; */
    /* box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); */
    padding: 20px;
}

.new-comer-elements {
    overflow-y: auto;
}

.new-comer-card {
    margin-top: 10px;
    border: 1px solid #ccc;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 20px;
    margin-bottom: 20px;
    /* background-color: #f9f9f9; */
}

.new-comer-card p {
    font-size: 18px;
    margin-bottom: 10px;
}

.card {
    /* flex: 1;  */
    /* border: 1px solid #ccc; */
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 20px;
    margin-bottom: 20px;
    background-color: #fff;
    /* padding:30px */
}

.card-content {
    display: flex;
    align-items: center;
    gap: 10px;
    /* margin-bottom: 10px; */
    padding: 20px
}

.select {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #f9f9f9;
}

.button {
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    cursor: pointer;
    transition: background-color 0.3s;
}

.button:hover {
    background-color: #0056b3;
}


.card-chart {
    flex-grow: 1;
    padding: 20px
}

.card-absent {
    /* flex: 1; Take up equal space within the container */
    border: 1px solid #ccc;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 20px;
    margin-bottom: 20px;
    background-color: #fff;
    height: 50%;
}

.card-content-absent {
    text-align: left;
}

.card-absent h2 {
    margin-bottom: 10px;
}

.card-absent p {
    font-size: 32px;
    font-weight: bold;
    /* color: #007bff; */
}
</style>