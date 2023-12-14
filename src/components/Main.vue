<script setup>
import { onMounted, ref, nextTick ,reactive} from "vue";


import GenerateButton from "./generator/GenerateButton.vue";
import PrivateKeyField from "./generator/PrivateKeyField.vue";
import PublicKeyField from "./generator/PublicKeyField.vue";

const worker = new Worker(new URL("./workers/worker.js", import.meta.url), {
  type: "module",
});

const publicKey = ref("");
const privateKey = ref("");

const start = ref("");
const end = ref("");

const loading = ref(false);

function update({ start: st, end: en }) {
  start.value = st;
  end.value = en;
}

function generateKeys() {
  loading.value = true;
  worker.postMessage({ start: start.value, end: end.value });
}

function invalid() {
  console.error("Invalid ethereum address");
}
function closeLoad(){
    loading.value = false;
}
onMounted(async () => {
  await nextTick();
  worker.onmessage = ({ data }) => {
    publicKey.value = data.publicKey;
    privateKey.value = data.privateKey;
    loading.value = false;
  };
  worker.onmessageerror = (e) => {
    loading.value = false;
    console.error(e);
  };
});

</script>

<template>

<!-- Button trigger modal -->
<div v-show="loading" class="loading-modal">
    <div class="loading-overlay"></div>
    <div class="loading-content">
      <div class="custom-loader"></div>
      <br>
      <p>Loading...</p>
      <span>It may take long time..</span>   <br>
      <button @click="closeLoad" class="btn btn-secondary " type="button">Cancel</button>
    </div>
  </div>

  <div v-show="loading">Loading..It may take a long time</div>
     <form class="col" @submit.prevent="generateKeys">
    <div>
      <GenerateButton @update="update" @invalid="invalid" />
    </div>
   
    <div>
      <PrivateKeyField :value="privateKey" :publickey="publicKey" />
    </div>
    <div>
      <PublicKeyField :value="publicKey" />
    </div>
  </form>
</template>
<style scoped>
  .loading-modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9999;
  }

  .loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
  }

  .loading-content {
    max-width: 300px;
    background-color: #fff;
    width: 100%;
    height: 300px;
    display: flex;
    justify-content: center;
    align-items: center;
 flex-direction: column;
    z-index: 999;
    border-radius: 8px;
  }
  .loading-content p{
    font-size: 1.5rem;
    font-weight: 500;
  }
  .loading-content span{
    font-size: 1rem;
    font-weight: 400;
    color: gray;
  }.custom-loader {
  width: 75px;
  height: 75px;
  display: grid;
  border:6px solid #0000;
  border-radius: 50%;
  border-color:#141414 #0000;
  animation: s6 1s infinite linear;
}
.custom-loader::before,
.custom-loader::after {    
  content:"";
  grid-area: 1/1;
  margin:3px;
  border:inherit;
  border-radius: 50%;
}
.custom-loader::before {
  border-color:#8C8C8C #0000;
  animation:inherit; 
  animation-duration: .5s;
  animation-direction: reverse;
}
.custom-loader::after {
  margin:8px;
}

@keyframes s6 { 
  100%{transform: rotate(1turn)}
}
  </style>