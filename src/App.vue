<script setup>
import { computed, onMounted, onBeforeUnmount, ref } from "vue";
import db from "./firebase";
import { ref as firebaseRef, onValue, set } from "firebase/database";

import Chart from "primevue/chart";
import Knob from "primevue/knob";
import Toast from "primevue/toast";
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
  return moisture.value < 40 && motorSignal.value == 2 ? "ON" : "OFF";
});

const chartDataForMoisture = ref();
const chartOptionsForMoisture = ref(null);

const setChartDataForMoisture = () => {
  return {
    labels: [`Độ ẩm: ${moisture.value}`],
    datasets: [
      {
        label: "Độ ẩm",
        backgroundColor: ["rgba(139, 92, 246, 0.2)"],
        borderColor: ["rgb(139, 92, 246)"],
        borderWidth: 5,
        data: [moisture.value],
      },
    ],
  };
};

const setChartOptionsForMoisture = () => {
  const documentStyle = getComputedStyle(document.documentElement);
  const surfaceBorder = documentStyle.getPropertyValue("--surface-border");

  return {
    maintainAspectRatio: false,
    aspectRatio: 0.8,
    plugins: {
      legend: {
        labels: {
          color: "black",
          font: {
            // family: 'Exo 2',
            size: 20,
            weight: "bold",
          },
        },
      },
    },
    scales: {
      x: {
        ticks: {
          color: "black",
          font: {
            weight: 500,
          },
        },
        grid: {
          display: false,
          drawBorder: false,
        },
      },
      y: {
        min: 0,
        max: 1023,
        ticks: {
          color: "black",
          stepSize: 400,
        },
        grid: {
          color: surfaceBorder,
          drawBorder: false,
        },
      },
    },
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
  showMessage(`Motor đã được ${motorStatus.value === "OFF" ? "Tắt" : "Bật"}`);
};

const handleAutomaticallyTurnOnMotor = () => {
  set(firebaseRef(db, "Servo/"), 2);
  showMessage("Motor đã được chuyển sang chế độ tự động bật tắt");
};

const showMessage = (message) => {
  toast.add({ severity: "success", summary: message, life: 1000 });
};

function isMobileDevice() {
  return /Android|webOS|iPhone/i.test(navigator.userAgent);
}

// Sử dụng isMobileDevice() để xác định isMobile
var isMobile = isMobileDevice();
</script>

<template>
  <!-- <div class="bg-cover bg-center h-screen bg-[url('https://images.pexels.com/photos/907274/pexels-photo-907274.jpeg?auto=compress&cs=tinysrgb&w=600')]"> -->
  <div class="bg-[#e1e2e4] text-black overflow-hidden">
    <div
      class="md:p-10 mx-5 border bg-[#FFFFFF] rounded-lg p-5 text-center md:text-5xl text-3xl font-bold"
    >
      Hệ thống điều khiển và giám sát nhiệt độ, độ ẩm
    </div>
    <div
      class="p-5 flex border justify-center mt-5 mx-5 bg-[#FFFFFF] rounded-lg"
    >
      <div class="*:md:text-2xl text-lg">
        <div>
          <b class="mr-[5px]">Trạm</b>:<b class="ml-[25px]"
            >Đường 3/2, Xuân Khánh, Ninh Kiều, Cần Thơ</b
          >
        </div>
        <div>
          <b class="mr-[5px]">Ngày</b>:<b class="ml-[25px]">{{
            currentTime.toLocaleDateString()
          }}</b>
        </div>
        <div>
          <b class="mr-[26px]">Giờ</b>:<b class="ml-[25px]">{{
            currentTime.toLocaleTimeString()
          }}</b>
        </div>
      </div>
    </div>
    <Toast />
    <div class="md:p-10 p-0 flex flex-col gap-5 -mt-5 -mr-10 -ml-5">
      <div class="flex justify-between -gap-[5px] md:flex-row flex-col">
        <div class="md:w-full flex flex-col bg-[#FFFFFF] border rounded-lg">
          <div class="flex justify-center gap-10">
            <Knob
              v-model="tempC"
              readonly
              :size="isMobile ? 150 : 300"
              :min="0"
              :max="100"
              valueTemplate="{value}&#8451;"
              textColor="black"
            />
            <Knob
              v-model="tempF"
              readonly
              :size="isMobile ? 150 : 300"
              :min="32"
              :max="212"
              valueTemplate="{value}&#8457;"
              textColor="black"
            />
          </div>
          <div class="flex justify-center p-4 md:text-2xl text-lg">
            <b class="mr-[5px]">Nhiệt độ</b>:<b class="ml-[15px]"
              >{{ tempC }}&#8451;</b
            >
          </div>
        </div>
        <div class="md:w-full flex flex-col">
          <div
            class="md:h-[400px] h-[300px] flex flex-col items-center justify-center mx-5 bg-[#FFFFFF] border rounded-lg"
          >
            <Knob
              v-model="moisture"
              readonly
              :size="isMobile ? 150 : 300"
              :min="0"
              :max="100"
              valueTemplate="{value}%"
              textColor="black"
            />
            <div class="flex justify-center p-4 md:text-2xl text-lg">
              <b>Độ ẩm</b>
            </div>
          </div>
        </div>
      </div>
      <div
        class="flex flex-col justify-between items-center gap-4 p-5 mr-5 border bg-[#FFFFFF] rounded-lg"
      >
        <div class="md:text-2xl text-lg font-bold">
          Trạng thái Motor: {{ motorStatus }}
          <span v-if="motorSignal == 2">(Tự động bật tắt)</span>
        </div>
        <div class="flex gap-5 pb-5">
          <button
            @click="handleToggleMotor"
            :class="
              motorStatus === 'ON'
                ? 'bg-red-500 hover:bg-red-400'
                : 'bg-green-500 hover:bg-green-400'
            "
            class="text-white font-bold md:text-xl text-lg py-3 px-5 rounded-lg"
          >
            {{ motorStatus === "ON" ? "Tắt Motor" : "Bật Motot" }}
          </button>
          <button
            @click="handleAutomaticallyTurnOnMotor"
            class="bg-cyan-500 hover:bg-cyan-400 text-white font-bold text-xl py-3 px-5 rounded-lg"
          >
            Tự động bật tắt
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
