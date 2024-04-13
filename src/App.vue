<script setup>
import { computed, onMounted, onBeforeUnmount, ref } from "vue";
import db from "./firebase";
import { ref as firebaseRef, onValue, set } from "firebase/database";

import Chart from 'primevue/chart';
import Knob from 'primevue/knob';
import Toast from 'primevue/toast';
import { useToast } from "primevue/usetoast";
const toast = useToast();

const moisture = ref(0);
const tempC = ref(0);
const tempF = ref(0);
const motorSignal = ref(2);
const currentTime = ref(new Date());

const updateCurrentTime = () => {
  currentTime.value = new Date();
};

const updateTimeInterval = setInterval(updateCurrentTime, 1000);
onBeforeUnmount(() => {
  clearInterval(updateTimeInterval);
});

const motorStatus = computed(() => {
  if (motorSignal.value == 1) return "ON";
  if (motorSignal.value == 0) return "OFF";
  return moisture.value >= 1000 && motorSignal.value == 2 ? "ON" : "OFF";
});

const chartDataForMoisture = ref();
const chartOptionsForMoisture = ref(null);

const setChartDataForMoisture = () => {
  const documentStyle = getComputedStyle(document.body);
  return {
    labels: [`Độ ẩm: ${moisture.value}`],
    datasets: [
      {

        label: 'Độ ẩm',
        backgroundColor: documentStyle.getPropertyValue('--pink-500'),
        borderColor: documentStyle.getPropertyValue('--pink-500'),
        data: [moisture.value],
      }
    ]
  };
};

const setChartOptionsForMoisture = () => {
  const documentStyle = getComputedStyle(document.documentElement);
  const textColor = documentStyle.getPropertyValue('--text-color');
  const textColorSecondary = documentStyle.getPropertyValue('--text-color-secondary');
  const surfaceBorder = documentStyle.getPropertyValue('--surface-border');

  return {
    maintainAspectRatio: false,
    aspectRatio: 0.8,
    plugins: {
      legend: {
        labels: {
          color: textColor
        }
      }
    },
    scales: {
      x: {
        ticks: {
          color: textColorSecondary,
          font: {
            weight: 500
          }
        },
        grid: {
          display: false,
          drawBorder: false
        }
      },
      y: {
        min: 0,
        max: 1023,
        ticks: {
          color: textColorSecondary,
          stepSize: 400,
        },
        grid: {
          color: surfaceBorder,
          drawBorder: false
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
    tempC.value = data;
    tempF.value = Math.round(data * 1.8 + 32);
  });

  onValue(moistureRef, (snapshot) => {
    const data = snapshot.val();
    moisture.value = data;
    chartDataForMoisture.value = setChartDataForMoisture();
  });

  onValue(motorSignalRef, (snapshot) => {
    const data = snapshot.val();
    motorSignal.value = data;
  });

  chartOptionsForMoisture.value = setChartOptionsForMoisture();
});

const handleToggleMotor = () => {
  set(firebaseRef(db, "Servo/"), motorStatus.value === "ON" ? 0 : 1);
  showMessage(`Motor đã được ${motorStatus.value === "ON" ? "Tắt" : "Bật"}`);
};

const handleAutomaticallyTurnOnMotor = () => {
  set(firebaseRef(db, "Servo/"), 2);
  showMessage("Motor đã được chuyển sang chế độ tự động bật tắt");
};

const showMessage = (message) => {
  toast.add({ severity: 'success', summary: message, life: 1000 });
};
</script>

<template>
  <!-- <div class="bg-cover bg-center h-screen bg-[url('https://images.pexels.com/photos/907274/pexels-photo-907274.jpeg?auto=compress&cs=tinysrgb&w=600')]"> -->
  <div class="h-screen bg-[] bg-gradient-to-r from-[#4FD2F7] to-[#F8EA52]">
    <div class="p-5 text-center text-5xl text-[#4b55e9] font-bold font-['Poppins']">
      Hệ thống điều khiển và giám sát nhiệt độ, độ ẩm
    </div>
    <div class="p-5 flex justify-center text-xl">
      <div>
        <div>Trạm: Đường 3/2, Xuân Khánh, Ninh Kiều, Cần Thơ</div>
        <div>Ngày: {{ currentTime.toLocaleDateString() }}</div>
        <div>Giờ: {{ currentTime.toLocaleTimeString() }}</div>
      </div>
    </div>
    <Toast />
    <div class="p-10 mt-2 flex flex-col gap-5">
      <div class="flex justify-between gap-4">
        <div class="w-full flex flex-col">
          <div class="flex justify-center gap-10">
            <Knob v-model="tempC" valueColor="#f97316" readonly :size="300" :min="0" :max="100"
              valueTemplate="{value}&#8451; " />
            <Knob v-model="tempF" valueColor="#f97316" readonly :size="300" :min="32" :max="212"
              valueTemplate="{value}&#8457; " />
          </div>
          <div class="flex justify-center p-4">Nhiệt độ: {{ tempC }}&#8451;</div>
        </div>
        <div class="w-full flex flex-col">
          <div class="h-[400px] flex justify-center">
            <Chart type="bar" :data="chartDataForMoisture" :options="chartOptionsForMoisture" />
          </div>
        </div>
      </div>
      <div class="flex flex-col justify-between items-center gap-4">
        <div class="text-2xl">
          Trạng thái Motor: {{ motorStatus }} <span v-if="motorSignal == 2">(Tự động bật tắt)</span>
        </div>
        <div class="flex gap-5">
          <button @click="handleToggleMotor"
            :class="motorStatus === 'ON' ? 'bg-red-500 hover:bg-red-400' : 'bg-green-500 hover:bg-green-400'"
            class="text-white font-bold text-xl py-3 px-5 rounded-lg">
            {{ motorStatus === "ON" ? "Tắt Motor" : "Bật Motot" }}
          </button>
          <button @click="handleAutomaticallyTurnOnMotor"
            class="bg-cyan-500 hover:bg-cyan-400 text-white font-bold text-xl py-3 px-5 rounded-lg">Tự động bật
            tắt</button>
        </div>
      </div>
    </div>
  </div>
</template>
