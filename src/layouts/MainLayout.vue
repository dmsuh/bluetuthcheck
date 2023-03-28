<template>
  <q-layout class="body" view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar
        class="flex-center"
      >
          <q-toolbar-title
            shrink
          >
            InnoTreker
          </q-toolbar-title>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { SafeArea } from 'capacitor-plugin-safe-area';
const check = SafeArea.getSafeAreaInsets().then(({ insets }) => {
  console.log(insets);
});

SafeArea.getStatusBarHeight().then(({statusBarHeight}) => {
  console.log(statusBarHeight, 'statusbarHeight');
})
</script>
