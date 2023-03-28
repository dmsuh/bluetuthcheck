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
        <div>
          <p>Данные из сервиса</p>
          <p>Сервис {{currentData.serviceId}}</p>
          <p>Характеристика для прослушки {{currentData.characteristicId}}</p>
          <p>{{currentData.value}}</p>
        </div>
        <q-list>
          <q-item v-for="(service,ind) in services.services" :key="service">
            <q-item-section>
              <p>{{service.uuid}}</p>
              <q-list>
                <q-item-section>
                  <q-item v-for="characteristics in services.services[ind].characteristics " :key="characteristics">
                    <p>characteristics + {{characteristics.uuid}}</p>
                    <q-btn @click="readDataFromCharacteristic(service.uuid,characteristics.uuid)" >Получить данные</q-btn>
                  </q-item>
                </q-item-section>
              </q-list>
            </q-item-section>
          </q-item>
        </q-list>
        <p>{{ services }}</p>
        <p>{{ result }}</p>
      </q-card-section>
    </q-card>
  </q-expansion-item>
</template>

<script setup lang="ts">

import {computed, ComputedRef, reactive, ref} from 'vue';
import {BleClient, BleServices} from '@capacitor-community/bluetooth-le';
const services:BleServices = reactive({ services:[]});
const isConnected = ref(false);
const currentData = reactive({value:{},characteristicId:'',serviceId:''})

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
    // connect to device, the onDisconnect callback is optional
    await BleClient.connect(deviceId, (deviceId) => onDisconnect(deviceId));
    isConnected.value=true

    services.services = await BleClient.getServices(deviceId);


  } catch (error) {

  }
}
const readDataFromCharacteristic = async (serviceId:string, characteristicId:string) => {
  try {
    const  value:any  = await BleClient.read(props.result.device.deviceId, serviceId, characteristicId );
    currentData.characteristicId=characteristicId;
    currentData.serviceId=serviceId;
    currentData.value = value;
  } catch (error) {
    console.error(error);
  }
}
function onDisconnect(deviceId: string): void {
  console.log(`device ${deviceId} disconnected`);
}
</script>
