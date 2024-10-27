<template>
  <div class="toast toast-right toast-top">
    <div class="badge badge-neutral"
      :class="{ 'badge-success': connection.connected, 'badge-error': !connection.connected }">
      {{ connection.connected ? "Connected" : "Disconnected" }}
    </div>
  </div>


  <!-- Creator's view for setting up the room -->
  <div v-if="!roomData.started && creator && !roomData.created"
    class="w-full h-full flex flex-col items-center max-w-sm mx-auto p-10">
    <div class="label w-full">
      <div class="label-text">Starting Money</div>
    </div>
    <input type="text" v-model="roomData.startingMoney" class="input input-bordered w-full text-center" />
    <div class="label w-full">
      <div class="label-text">Pass Go Money</div>
    </div>
    <input type="text" v-model="roomData.passGoMoney" class="input input-bordered w-full text-center" />

    <button @click="createRoom" class="btn btn-primary w-full mt-5">Create Room</button>
  </div>
  <!-- Player name input for joining the game -->
  <div v-else-if="!playerJoined" class="w-full h-full flex flex-col items-center max-w-sm mx-auto p-10">
    <div class="label w-full">
      <div class="label-text">Enter your name</div>
    </div>
    <input type="text" v-model="playerName" class="input input-bordered w-full text-center" />
    <button @click="joinGame" class="btn btn-primary w-full mt-5">Join Game</button>
  </div>



  <div class="w-full h-full flex flex-col items-center max-w-sm mx-auto p-10"
    v-else-if="roomData.created && !roomData.started">

    <div v-for="player in roomData.players" :key="player.name" class="w-full mt-5">
      <div class="flex justify-between items-center border rounded-lg p-2 py-3 input-bordered">
        <span>{{ player.name }}</span>
        <span>${{ player.money }}</span>
      </div>
    </div>
    <button @click="startGame" class="btn btn-primary w-full mt-5" v-if="!roomData.started && creator">Start
      Game</button>
    <div v-else class="text-xs mt-5">
      Waiting for other players...
    </div>
  </div>


  <!-- Game view after joining -->
  <div v-else-if="roomData.created" class="w-full h-full flex flex-col items-center max-w-sm mx-auto p-10">
    <div class="w-full  flex justify-between items-center">
      <p class="text-sm">Balance</p>
      <p class="text-xl rounded-md p-2 bg-neutral">${{ playerData?.money }}</p>
    </div>
    <div class="divider"></div>

    <button class="btn btn-neutral btn-circle mt-5 w-36 h-36 p-1" @click="payDialog = true">
      <div class="flex flex-col bg-primary w-full h-full justify-center items-center rounded-full text-white">
        <span class="text-2xl">Pay</span>
      </div>
    </button>

    <div
      class="w-full h-5/6 flex flex-col items-center absolute border rounded-t-2xl input-bordered bg-[#161b20] transition-all duration-300"
      :class="payDialog ? 'bottom-0' : 'bottom-[-100%]'">
      <button @click="payDialog = false" class="btn btn-circle btn-neutral btn-sm absolute top-2 right-2">
        X
      </button>

      <div class="w-full h-full flex flex-col items-center p-5 gap-3 pt-14">
        <div class="grid grid-cols-2 gap-2 w-5/6">
          <div @click="selectedPlayer = '923457adfgkshj'"
            class="w-full border border-neutral p-2 rounded-md text-center h-fit"
            :class="selectedPlayer === '923457adfgkshj' ? 'bg-primary text-black' : ''">
            <span class="text-sm text-center w-full">Bank</span>
          </div>
          <div v-for="player in roomData.players.filter(player => player.id !== currentPlayerId)" :key="player.name"
            @click="selectedPlayer = player.id" class="w-full border border-neutral p-2 rounded-md text-center h-fit"
            :class="selectedPlayer === player.id ? 'bg-primary text-black' : ''">
            <span class="text-sm text-center w-full">{{ player.name }}</span>
          </div>
        </div>
        <input type="text" v-model="amount" class="input input-bordered text-center mt-auto min-h-14 w-5/6" readonly />

        <div class="grid grid-cols-3 w-5/6 gap-2 justify-items-center ">
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '1'">1</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '2'">2</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '3'">3</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '4'">4</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '5'">5</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '6'">6</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '7'">7</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '8'">8</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '9'">9</button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount.slice(0, -1)">
            <Delete class="h-8 w-8" />
          </button>
          <button class="btn btn-neutral h-full w-full aspect-square text-2xl" @click="amount = amount + '0'">
            0
          </button>
          <button class="btn btn-primary h-full w-full aspect-square text-2xl" @click="pay">
            <Check class="h-8 w-8" />
          </button>
        </div>
      </div>


    </div>

    <!-- New button to toggle bank request popup -->
    <button @click="toggleBankRequestPopup" class="btn btn-secondary w-full">
      Bank Requests ({{ bankRequests.length }})
    </button>

    <!-- New popup for bank requests -->
    <div v-if="showBankRequestPopup" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white p-6 rounded-lg max-w-md w-full max-h-[80vh] overflow-y-auto">
        <h2 class="text-xl font-bold mb-4">Bank Requests</h2>
        <div v-if="bankRequests.length === 0" class="text-center text-gray-500">
          No pending bank requests
        </div>
        <div v-for="request in bankRequests" :key="request.id" class="mb-4 p-4 border rounded-lg">
          <p><strong>{{ getPlayerName(request.player_id) }}</strong> requests ${{ request.amount }}</p>
          <div class="flex justify-between mt-2">
            <button @click="respondToBankRequest(request.id, true)" class="btn btn-sm btn-success">Approve</button>
            <button @click="respondToBankRequest(request.id, false)" class="btn btn-sm btn-error">Reject</button>
          </div>
        </div>
        <button @click="toggleBankRequestPopup" class="btn btn-neutral w-full mt-4">Close</button>
      </div>
    </div>

  </div>

  <div
    class="m-3 border rounded-lg p-4 text-center toast toast-center toast-top input-bordered text-sm bg-neutral drop-in-animation"
    v-if="alertMessage">
    {{ alertMessage }}
  </div>


</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import { useRoute } from 'vue-router';
// import { socket } from '../socket';
import { io } from "socket.io-client";
import { Delete } from 'lucide-vue-next';
import { Check } from 'lucide-vue-next';


const route = useRoute();
const roomId = route.params.roomId;
const creator = route.query.creator === 'true';

const alertMessage = ref("");
const roomData = ref({});
const playerName = useLocalStorage("playerName", "");
const playerJoined = ref(false);
const currentPlayerId = ref("");
const selectedPlayer = ref("");
const amount = ref("");

const payDialog = ref(false);
const bankRequests = ref([]);
const showBankRequestPopup = ref(false);

const playerData = computed(() => {
  return roomData.value.players.find(player => player.id === currentPlayerId.value);
});


const socket = io(import.meta.env.VITE_SERVER_URL,);


const setAlertMessage = (message) => {
  alertMessage.value = message;
  setTimeout(() => {
    alertMessage.value = "";
  }, 3000);
};

const connection = reactive({
  connected: false,
});

socket.on("connect", () => {
  currentPlayerId.value = socket.id;
  connection.connected = true;
  socket.emit("join", roomId);
});

socket.on("roomData", (data) => {
  roomData.value = data;
  console.log(roomData.value);
});

socket.on("error", (message) => {
  setAlertMessage(message);
});

socket.on("user_joined", (message) => {
  setAlertMessage(message);
});

socket.on("newRoomData", (data) => {
  console.log("newRoomData", data);
  roomData.value = data;
});

socket.on("paymentRecieved", (data) => {
  setAlertMessage(`${data.from} paid you ${data.amount}`);
});

socket.on("paymentSent", (data) => {
  setAlertMessage(`You paid ${data.to} ${data.amount}`);
});

const createRoom = () => {
  socket.emit("setRoomData", {
    room: roomId,
    startingMoney: roomData.value.startingMoney,
    passGoMoney: roomData.value.passGoMoney,
  });
};

const joinGame = () => {
  if (playerName.value.trim() === "") {
    setAlertMessage("Please enter a name");
    return;
  }
  socket.emit("joinGame", {
    room: roomId,
    player_name: playerName.value,
  });
  playerJoined.value = true; 1
};

const startGame = () => {
  socket.emit("startGame", roomId);
};

const pay = () => {
  console.log(amount.value, selectedPlayer.value)
  if (amount.value === "") {
    setAlertMessage("Please enter an amount");
    return;
  }

  if (selectedPlayer.value === "") {
    setAlertMessage("Please select a player");
    return;
  }

  if (amount.value > playerData.value.money) {
    setAlertMessage("You don't have enough money");
    return;
  }

  socket.emit("pay", {
    room: roomId,
    amount: amount.value,
    to: selectedPlayer.value,
    from: currentPlayerId.value,
  });

  amount.value = "";
  selectedPlayer.value = "";
  payDialog.value = false;
};

const toggleBankRequestPopup = () => {
  showBankRequestPopup.value = !showBankRequestPopup.value;
};

const getPlayerName = (playerId) => {
  const player = roomData.value.players.find(p => p.id === playerId);
  return player ? player.name : 'Unknown Player';
};

const respondToBankRequest = (requestId, approved) => {
  socket.emit('respondToBankRequest', {
    room: roomData.value.id,
    request_id: requestId,
    player_id: currentPlayer.value.id,
    approved: approved
  });
};

const requestFromBank = () => {
  const amount = prompt('Enter the amount to request from the bank:');
  if (amount && !isNaN(amount)) {
    socket.emit('requestFromBank', {
      room: roomData.value.id,
      player_id: currentPlayer.value.id,
      amount: parseInt(amount)
    });
  }
};

onMounted(() => {
  if (creator) {
    roomData.value = {
      id: roomId,
      started: false,
      created: false,
      startingMoney: 1500,
      passGoMoney: 200,
      creator: true,
      players: [],
    };
  }

  socket.on('newBankRequest', (request) => {
    bankRequests.value.push(request);
  });

  socket.on('bankRequestApproved', (request) => {
    const index = bankRequests.value.findIndex(r => r.id === request.id);
    if (index !== -1) {
      bankRequests.value.splice(index, 1);
    }
    alert(`Bank request for $${request.amount} has been approved for ${getPlayerName(request.player_id)}`);
  });

  socket.on('bankRequestRejected', (request) => {
    const index = bankRequests.value.findIndex(r => r.id === request.id);
    if (index !== -1) {
      bankRequests.value.splice(index, 1);
    }
    alert(`Bank request for $${request.amount} has been rejected for ${getPlayerName(request.player_id)}`);
  });

  socket.on('bankRequestUpdated', (request) => {
    const index = bankRequests.value.findIndex(r => r.id === request.id);
    if (index !== -1) {
      bankRequests.value[index] = request;
    }
  });
});
</script>

<style lang="scss" scoped>
.grc {}
</style>
