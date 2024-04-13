<script setup>
import { computed, onMounted, ref } from "vue";
import db from "./firebase";
import { ref as firebaseRef, onValue, set } from "firebase/database";

import Chart from 'primevue/chart';

const moisture = ref(0);
const temp = ref(0);
const motorSignal = ref(2);

const motorStatus = computed(() => {
  if (motorSignal.value == 1) return "ON";
  if (motorSignal.value == 0) return "OFF";
  return moisture.value >= 1000 && motorSignal.value == 2 ? "ON" : "OFF";
});

const chartDataForTemp = ref();
const chartDataForMoisture = ref();
const chartOptions = ref(null);

const setChartDataForTemp = () => {
  const documentStyle = getComputedStyle(document.body);
  return {
    labels: ['A', 'B'],
    datasets: [
      {
        data: [temp.value, 100 - temp.value],
        backgroundColor: [documentStyle.getPropertyValue('--orange-500'), documentStyle.getPropertyValue('--gray-500')],
        hoverBackgroundColor: [documentStyle.getPropertyValue('--orange-400'), documentStyle.getPropertyValue('--gray-400')]
      }
    ]
  };
};

const setChartDataForMoisture = () => {
  const documentStyle = getComputedStyle(document.body);
  return {
    labels: ['A', 'B'],
    datasets: [
      {
        data: [moisture.value, 1023 - moisture.value],
        backgroundColor: [documentStyle.getPropertyValue('--orange-500'), documentStyle.getPropertyValue('--gray-500')],
        hoverBackgroundColor: [documentStyle.getPropertyValue('--orange-400'), documentStyle.getPropertyValue('--gray-400')]
      }
    ]
  };
};

const setChartOptions = () => {
  const documentStyle = getComputedStyle(document.documentElement);
  const textColor = documentStyle.getPropertyValue('--text-color');

  return {
    plugins: {
      legend: {
        labels: {
          cutout: '10%',
          color: textColor
        }
      }
    }
  };
};

onMounted(() => {

  const moistureRef = firebaseRef(db, "Moisture");
  const tempRef = firebaseRef(db, "Temp");
  const motorSignalRef = firebaseRef(db, "Servo");

  onValue(tempRef, (snapshot) => {
    const data = snapshot.val();
    temp.value = data;
  });

  onValue(moistureRef, (snapshot) => {
    const data = snapshot.val();
    moisture.value = data;
  });

  onValue(motorSignalRef, (snapshot) => {
    const data = snapshot.val();
    motorSignal.value = data;
  });

  chartDataForTemp.value = setChartDataForTemp();
  chartDataForMoisture.value = setChartDataForMoisture();
  chartOptions.value = setChartOptions();
});

const handleToggleMotor = () => {
  set(firebaseRef(db, "Servo/"), motorStatus.value === "ON" ? 0 : 1);
};

const handleAutomaticallyTurnOnMotor = () => {
  set(firebaseRef(db, "Servo/"), 2);
};

</script>

<template>
  <!-- <div class="bg-cover bg-center h-screen bg-[url('https://images.pexels.com/photos/907274/pexels-photo-907274.jpeg?auto=compress&cs=tinysrgb&w=600')]"> -->
  <div class="h-screen bg-[] bg-gradient-to-r from-[#4FD2F7] to-[#F8EA52]">
    <div class="p-5 text-center text-4xl text-[#4b55e9] font-bold font-['Poppins']">
      Hệ thống điều khiển và giám sát nhiệt độ, độ ẩm
    </div>
    <div class="p-10 flex flex-col gap-5">
      <div class="flex justify-between gap-4">
        <div class="w-full flex flex-col">
          <div class="flex justify-center">
            <Chart type="doughnut" :data="chartDataForTemp" :options="chartOptions" />
          </div>
          <div class="flex justify-center p-4">Nhiệt độ: {{ temp }}</div>
        </div>
        <div class="w-full flex flex-col">
          <div class="flex justify-center">
            <Chart type="doughnut" :data="chartDataForMoisture" :options="chartOptions" />
          </div>
          <div class="flex justify-center p-4">Độ ẩm: {{ moisture }}</div>
        </div>
      </div>
      <div class="flex flex-col justify-between items-center gap-4">
        <div class="text-2xl">
          Trạng thái Motor: {{ motorStatus }} <span v-if="motorSignal == 2">(Tự động bật tắt)</span>
        </div>
        <div class="flex gap-5">
          <button @click="handleToggleMotor" :class="motorStatus === 'ON' ? 'bg-red-500 hover:bg-red-400' : 'bg-green-500 hover:bg-green-400'" class="text-white font-bold text-xl py-3 px-5 rounded-lg">
            {{ motorStatus === "ON" ? "Tắt Motor" : "Bật Motot" }}
          </button>
          <button @click="handleAutomaticallyTurnOnMotor" class="bg-cyan-500 hover:bg-cyan-400 text-white font-bold text-xl py-3 px-5 rounded-lg">Tự động bật tắt</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* body {
  background-image: url("https://images.pexels.com/photos/907274/pexels-photo-907274.jpeg?auto=compress&cs=tinysrgb&w=600");
  background-repeat: no-repeat;
  width: 100%;
} */
</style>