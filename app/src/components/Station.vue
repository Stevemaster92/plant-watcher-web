<template>
    <div class="ring-1 bg-white ring-gray-200 rounded-md p-5">
        <Notification
            :class="{ 'opacity-1': success, 'opacity-0': !success }"
            class="fixed right-10 transition-opacity duration-500"
            message="Config updated"
            type="success"
        />
        <Notification
            :class="{ 'opacity-1': error, 'opacity-0': !error }"
            class="fixed right-10 transition-opacity duration-500"
            message="Failed to update config"
            type="error"
        />

        <h2 class="custom-title">Station {{ station.stationId }}</h2>

        <Config v-if="isLoggedIn" @save-config="saveConfig" :config="station.config" />

        <div v-for="(sensor, index) in sensorData" :key="index">
            <Sensor
                v-if="sensor"
                :sensorId="sensor[0].sensorId"
                :sensorData="sensor"
                :measureInterval="station.config.measureInterval"
            />
        </div>
    </div>
</template>

<script>
import Config from "./Config.vue";
import Sensor from "./Sensor.vue";
import Notification from "./Notification.vue";

export default {
    components: {
        Config,
        Sensor,
        Notification,
    },
    props: {
        station: {
            type: Object,
            default: null,
        },
    },
    data() {
        return {
            success: false,
            error: false,
            sensorData: Array,
            apiUrl: import.meta.env.VITE_API_URL,
            isLoggedIn: false,
        };
    },
    mounted() {
        this.$auth.onAuthStateChanged((user) => {
            this.isLoggedIn = user !== null;
        });

        this.$axios.get(`${this.apiUrl}/stations/${this.station.stationId}/sensors`).then(async (res) => {
            const sensors = await res.data;

            // Create array of arrays of sensor data.
            this.sensorData = sensors.reduce((prev, sensor) => {
                if (!prev[sensor.sensorId]) {
                    prev[sensor.sensorId] = [];
                }
                prev[sensor.sensorId].push(sensor);

                return prev;
            }, []);
        });
    },
    methods: {
        saveConfig() {
            this.$axios
                .post(`${this.apiUrl}/stations`, this.station)
                .then((res) => {
                    if (res.status === 200) {
                        this.success = true;
                    } else {
                        this.error = true;
                    }

                    this.clearFlags();
                })
                .catch(() => {
                    this.error = true;
                    this.clearFlags();
                });
        },
        clearFlags() {
            setInterval(() => {
                this.success = false;
                this.error = false;
            }, 2000);
        },
    },
};
</script>

<style></style>
