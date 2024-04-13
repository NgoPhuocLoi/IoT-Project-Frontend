<script setup>
import { computed, onMounted, ref } from "vue";
import db from "./firebase";
import { ref as firebaseRef, onValue, set } from "firebase/database";

const moisture = ref(0);
const temp = ref(0);
const motorSignal = ref(2);

const motorStatus = computed(() => {
  if (motorSignal.value == 1) return "ON";
  if (motorSignal.value == 0) return "OFF";
  return moisture.value >= 1000 && motorSignal.value == 2 ? "ON" : "OFF";
});

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
});

const handleToggleMotor = () => {
  set(firebaseRef(db, "Servo/"), motorStatus.value === "ON" ? 0 : 1);
};

const handleAutomaticallyTurnOnMotor = () => {
  set(firebaseRef(db, "Servo/"), 2);
};
</script>

<template>
  <h1>Project</h1>
  <div>
    <div>Moisture: {{ moisture }}</div>
    <div>Temp: {{ temp }}</div>
  </div>

  <div>
    <div>
      Motor Status: {{ motorStatus }}
      <span v-if="motorSignal == 2">(Tự động bật tắt)</span>
    </div>
  </div>

  <button @click="handleToggleMotor">
    {{ motorStatus === "ON" ? "Tắt Motor" : "Bật Motot" }}
  </button>
  <button @click="handleAutomaticallyTurnOnMotor">Tự động bật tắt</button>
</template>
