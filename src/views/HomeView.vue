<template>
  <div class="w-full h-full flex flex-col items-center justify-center gap-4 max-w-sm mx-auto p-10">
    <input type="text" v-model="roomId" class="input input-bordered w-full text-center" />
    <button @click="joinRoom" class="btn w-full" :class="roomId.length === 5 ? 'btn-primary' : 'btn-neutral'"
      :disabled="roomId.length !== 5">Join
      Room</button>
    <input type="text" v-model="generatedRoomId" readonly class="input input-bordered w-full text-center" />
    <button @click="createNewRoom" class="btn btn-primary w-full">Create New</button>
    <div
      class="m-3 border rounded-lg p-4 text-center toast toast-center toast-top input-bordered text-sm bg-neutral drop-in-animation"
      v-if="alertMessage">
      {{ alertMessage }}
    </div>
  </div>
</template>

<script setup>
import { v4 as uuidv4 } from "uuid";
import axios from "axios";


const alertMessage = ref("");
const roomId = ref("");
const generatedRoomId = ref(uuidv4().slice(0, 5));
const router = useRouter();
const userId = useLocalStorage("userId", uuidv4());

onMounted(() => {
  if (userId.value === "") {
    userId.value = uuidv4();
  }
});


const setAlertMessage = (message) => {
  alertMessage.value = message;
  setTimeout(() => {
    alertMessage.value = "";
  }, 3000);
};

const joinRoom = () => {
  router.push(`/game/${roomId.value}`);
};

const createNewRoom = async () => {
  console.log(import.meta.env.VITE_SERVER_URL);
  const response = await axios.post(`${import.meta.env.VITE_SERVER_URL}/create_room`,
    { id: generatedRoomId.value },
  );

  if (response.data.available) {
    router.push(`/game/${generatedRoomId.value}?creator=true`);
  } else {
    setAlertMessage("Room already exists");
  }
};
</script>

<style lang="scss" scoped>
.drop-in-animation {
  animation: drop-in 0.3s ease-in-out;
}

@keyframes drop-in {
  0% {
    top: -100%;
  }

  100% {
    top: 0;
  }
}
</style>
