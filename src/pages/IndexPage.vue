<template>
  <q-page>
    <div
      v-if="$q.platform.is.capacitor"
      class="absolute fit column flex-center no-wrap"
    >
      <div class="column flex-center">
        <q-icon
          name="fab fa-bluetooth"
          :color="ble_enabled ? 'primary' : 'gray'"
          size="lg"
          class="q-ma-md"
        />
        <q-btn
          label="Личный кабинет"
          class="q-ma-md"
          :disable="true"
        />
        <q-space/>
        Bluetooth {{ ble_enabled ? 'Включен' : 'Выключен' }}
        <q-btn
          :label="scanning ? 'Поиск...' : 'Начать поиск'"
          color="primary"
          :loading="scanning"
          :disable="!ble_enabled"
          class="q-ma-md"
          @click="scan"
        />
      </div>

      <div class="scroll">
        <q-list
          v-if="results.length"
          bordered
          separator
          class="rounded-borders"
        >
          <ResultList
            v-for="result in filtered_results"
            :key="result.device.deviceId"
            :result="result"
          />
        </q-list>
        <span
          v-else-if="!scanning && tried_scanning"
          class="q-ma-lg text-subtitle1"
        >Устройств не найдено</span
        >
      </div>
      <q-btn
        v-if="results.length"
        :label="see_all ? 'Спрятать незнакомые девайсы' : 'Показать все'"
        flat
        color="primary"
        @click="see_all = !see_all"
      />
    </div>
  </q-page>
</template>

<script setup lang="ts">

import {
  BleClient,
  ScanResult
} from '@capacitor-community/bluetooth-le'
import {computed, ComputedRef, onMounted, ref} from 'vue';
import ResultList from 'components/ResultList.vue';

const results = ref<ScanResult[]>([])
const scanning = ref(false)
const tried_scanning = ref(false)
const ble_enabled = ref(false)
const see_all = ref(false)
const error = ref('')

const filtered_results: ComputedRef<ScanResult[]> = computed((): ScanResult[] => {
  if (see_all.value) return results.value
  else return results.value.filter(result => result.localName)
});
const init = async () => {
  try {
    await BleClient.initialize()
    ble_enabled.value = await BleClient.isEnabled()
    await BleClient.startEnabledNotifications((enabled: boolean) => {
      ble_enabled.value = enabled
    })
  } catch (err) {
    error.value = (err as Error).message
  }
}
const scan = async () => {
  results.value = []
  scanning.value = true
  tried_scanning.value = true
  try {
    await BleClient.requestLEScan({}, result => {
      results.value.push(result)
    })
    setTimeout(() => {
      void BleClient.stopLEScan().then(() => {
        scanning.value = false
      })
    }, 10000)
  } catch (err) {
    error.value = (err as Error).message
  }
}
onMounted(init)


</script>
