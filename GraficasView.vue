<script lang="ts" setup>
import { ref } from 'vue'
import GraficaHijo from '../components/GraficasHijos.vue'

let gap: number = 0
const urlGraficas: string =
  import.meta.env.VITE_APP_ROOT_API + '/graficas'
let graficasF = ref<any>([
  {
    url: urlGraficas + `?gap=${gap}&type=sesion`,
    type: 'sesion'
  },
  {
    url: urlGraficas + `?gap=${gap}&type=lecesc`,
    type: 'lecesc'
  },
  {
    url: urlGraficas + `?gap=${gap}&type=secuencial`,
    type: 'secuencial'
  }
])

function updateCharts() {
  graficasF.value = [
    {
      url: urlGraficas + `?gap=${gap}&type=sesion`,
      type: 'sesion'
    },
    {
      url: urlGraficas + `?gap=${gap}&type=lecesc`,
      type: 'lecesc'
    },
    {
      url: urlGraficas + `?gap=${gap}&type=secuencial`,
      type: 'secuencial'
    }
  ]
}

async function backGap() {
  if (gap == 168) {
  } else {
    gap += 1
    updateCharts()
  }
}

async function forwardGap() {
  if (gap == 0) {
  } else {
    gap -= 1
    updateCharts()
  }
}
</script>
<template>
  <v-container fluid>
    <v-row justify="center" no-gutters>
      <v-col cols="auto" class="mx-auto" v-for="item in graficasF">
        <GraficaHijo :url="item.url" :type="item.type"/>
      </v-col>
    </v-row>
    <v-row justify="center">
      <v-col cols="auto">
        <v-btn
          @click="backGap"
          class="mr-7 mt-4"
          color="blue-grey-darken-2"
          elevation="5"
          size="x-large"
          ><v-icon>mdi-arrow-left-bold</v-icon></v-btn
        >
      </v-col>
      <v-col cols="auto">
        <v-btn
          @click="forwardGap"
          class="ml-7 mt-4"
          color="blue-grey-darken-2"
          elevation="5"
          size="x-large"
          ><v-icon>mdi-arrow-right-bold</v-icon></v-btn
        >
      </v-col>
    </v-row>
  </v-container>
</template>
<style scoped></style>
