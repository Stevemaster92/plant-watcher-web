<template>
    <div>
        <h3 class="custom-subtitle">Sensor {{ sensorId }}</h3>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
            <div>
                <h4 class="chart-title">Current moisure</h4>
                <canvas class="max-h-96" :id="'doughnut-chart-' + sensorId"></canvas>
            </div>
            <div>
                <h4 class="chart-title">Moisure of last {{ chartTitleValue }} hours</h4>
                <canvas :id="'line-chart-' + sensorId"></canvas>
            </div>
        </div>
    </div>
</template>

<script>
import Chart from "chart.js/auto";

export default {
    props: {
        sensorId: Number,
        sensorData: Array,
        measureInterval: Number, // in ms
        measurePoints: {
            type: Number,
            default: 24,
        },
    },
    methods: {
        getLatest() {
            return this.sensorData.slice(this.sensorData.length - this.measurePoints, this.sensorData.length);
        },
        getMoistureColor(moisture) {
            if (moisture > 80) {
                return "rgba(39, 174, 96, 1)"; // Green
            }
            if (moisture > 50) {
                return "rgba(45, 156, 219, 1)"; // Blue
            }
            if (moisture > 20) {
                return "rgba(242, 201, 76, 1)"; // Yellow
            }

            return "rgba(235, 87, 87, 1)"; // Red
        },
        fillData() {
            // Get sensor data of the last 24 hours.
            const latestSensors = this.getLatest();
            const smallOptions = {
                indexAxis: "y",
                scales: {
                    x: {
                        beginAtZero: true,
                        max: 100,
                    },
                },
            };
            const normalOptions = {
                scales: {
                    y: {
                        beginAtZero: true,
                        max: 100,
                    },
                },
            };

            // Line chart
            new Chart(document.getElementById(`line-chart-${this.sensorId}`), {
                type: "line",
                data: {
                    labels: latestSensors.map((sensor) => new Date(sensor.createdAt).toLocaleString()),
                    datasets: [
                        {
                            label: "Moisture",
                            backgroundColor: "rgba(79, 70, 229, 1)",
                            borderColor: "rgba(79, 70, 229, 1)",
                            data: latestSensors.map((sensor) => sensor.moisture),
                            tension: 0.1,
                            tooltip: {
                                callbacks: {
                                    label: (context) => `${context.dataset.label}: ${context.parsed.y} %`,
                                },
                            },
                        },
                    ],
                },
                options:
                    // Change the chart to vertical on small displays.
                    window.innerWidth <= 425 ? smallOptions : normalOptions,
            });

            // Doughnut chart
            const lastSensor = latestSensors[latestSensors.length - 1];
            const moistureColor = this.getMoistureColor(lastSensor.moisture);

            new Chart(document.getElementById(`doughnut-chart-${this.sensorId}`), {
                type: "doughnut",
                data: {
                    labels: ["Moisure"],
                    datasets: [
                        {
                            backgroundColor: [moistureColor, "rgba(27,31,35,.05)"],
                            hoverBackgroundColor: [moistureColor, "rgba(27,31,35,.05)"],
                            hoverOffset: 10,
                            data: [lastSensor.moisture, 100 - lastSensor.moisture],
                            tooltip: {
                                callbacks: {
                                    label: (context) => `${context.label}: ${context.parsed} %`,
                                },
                            },
                        },
                    ],
                },
            });
        },
    },
    mounted() {
        this.fillData();
    },
    computed: {
        chartTitleValue() {
            // Return value in hours.
            return (this.measureInterval * this.measurePoints) / 3600000;
        },
    },
};
</script>

<style scoped>
.chart-title {
    @apply text-lg text-center italic mb-2;
}
</style>
