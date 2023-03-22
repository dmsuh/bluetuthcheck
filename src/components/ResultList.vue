<template>
  <q-expansion-item group="results">
    <template v-slot:header>
      <q-item-section>
        {{ result.localName || result.device.deviceId }}
      </q-item-section>
    </template>

    <q-card style="max-width: 100%">
      <q-card-section>
        <q-btn
          @click="connection(result.device.deviceId)"
        >
          Подключиться
        </q-btn>
        <p>{{ services }}</p>
        <p>{{ result }}</p>
      </q-card-section>
    </q-card>
  </q-expansion-item>
</template>

<script setup lang="ts">

import {computed, ComputedRef, reactive, ref} from 'vue';
import {BleClient} from "@capacitor-community/bluetooth-le";
const services = reactive({value:{}});
const isConnected = ref(false)

const signal_color: ComputedRef<string> = computed(
    (): string => {
      const latency = this.result.rssi * -1
      let color = 'red'
      if (latency < 70) color = 'green'
      else if (latency < 90) color = 'yellow'
      return color})
const props = defineProps(['result'])
const connection = async (deviceId: string): Promise<void> => {
  try {
    await BleClient.initialize();
    services.value={};

    // connect to device, the onDisconnect callback is optional
    await BleClient.connect(deviceId, (deviceId) => onDisconnect(deviceId));

    services.value = await BleClient.getServices(deviceId);


  } catch (error) {

  }
}
function onDisconnect(deviceId: string): void {
  console.log(`device ${deviceId} disconnected`);
}
</script>
