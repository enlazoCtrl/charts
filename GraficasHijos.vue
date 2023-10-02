<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  BarElement,
  CategoryScale,
  LinearScale,
  type ChartData
} from 'chart.js'
import { Bar } from 'vue-chartjs'
import Pivot from 'quick-pivot'
import type { IChartObject } from '@/interfaces/IChartObject'
import type { IGrafica } from '@/interfaces/IGrafica'

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend)
ChartJS.defaults.color = 'white'

const windowWidth = ref<number>(window.innerWidth)
let widthChartStyle = ref<number>(550)
let heightChartStyle = ref<number>(247)
let fullScreenWidth: number = 0
const router = useRouter()
let initalDataSet: string[][] = [['cuando', 'quien', 'cuanto']]
let pivot: any
let itemReactive = ref<ChartData<'bar'>>({
  datasets: []
})
let temporizador: any = null
let label: number = 0
const props = defineProps<{
  url: string
  type: string
}>()
const chart = ref<HTMLInputElement | null>(null)

watch(
  () => props.url,
  async (newUrl) => {
    await getDataset(newUrl)
  },
  { immediate: true }
)

watch(windowWidth, () => {
  handleResize()
})

function handleResize() {
  windowWidth.value = window.innerWidth
  if (windowWidth.value === fullScreenWidth) {
    widthChartStyle.value = 550
    heightChartStyle.value = 247
  } else {
    widthChartStyle.value = 426
    heightChartStyle.value = 213
  }
}

const options = ref({
  responsive: true,
  onresize: () => {
    handleResize()
  },
  scales: {
    x: {
      stacked: true
    },
    y: {
      stacked: true
    }
  },
  plugins: {
    legend: {
      display: false
    }
  },
  onClick: (e: any, element: any) => {
    // let chartR = Object.assign({}, chart.value) as Object as { chart: ChartJS }
    // const points = chartR.chart.getElementsAtEventForMode(e, 'nearest', { intersect: true }, false)
    if (element.length) {
      const firstPoint = element[0]
      label = itemReactive.value.labels![firstPoint.index] as number
      //label = chartR.chart.data.labels![firstPoint.index] as number
      router.push({ name: 'detalleConsumo', params: { grafica: props.type, label } })
    }
  }
})

let object: IChartObject = {
  labels: [''],
  datasets: [
    {
      label: '',
      backgroundColor: '',
      barThickness: 0,
      borderRadius: 0,
      data: [{}]
    }
  ]
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
  fullScreenWidth = window.innerWidth
  temporizador = setInterval(paintChart, 300000)
  chart.value?.focus
})

onUnmounted(() => {
  clearInterval(temporizador)
  window.removeEventListener('resize', handleResize)
})

function paintChart() {
  getDataset(props.url)
}

async function getDataset(url: string) {
  try {
    const response = await axios.get<IGrafica[]>(url)
    readyChartDataSet(response.data, object)
    emptyObject()
  } catch (error) {
    console.log(error)
    throw error
  }
}

function emptyObject(): void {
  object = {
    labels: [''],
    datasets: [
      {
        label: '',
        backgroundColor: '',
        barThickness: 0,
        borderRadius: 0,
        data: [{}]
      }
    ]
  }
}

function dameColorAleatorio(): string {
  let simbolos: string, color: string
  simbolos = '0123456789ABCDEF'
  color = '#'
  for (let i = 0; i < 6; i++) {
    color = color + simbolos[Math.floor(Math.random() * 16)]
  }
  return color
}

function readyChartDataSet(data: any, chart: IChartObject): void {
  initalDataSet = [['cuando', 'quien', 'cuanto']]
  data.forEach((element: { cuando: string; usu: string; cpu: number }) => {
    const object: any[] = [element.cuando, element.usu, element.cpu]
    //Preparamos el array que hay que pivotar que posteriormente se pintará en la gráfica
    initalDataSet.push(object)
  })

  //Creamos el objeto de la librería quick-pivot que formatea los datos como deseamos
  pivot = new Pivot(initalDataSet, ['quien'], ['cuando'], 'cuanto', 'sum')

  //agregamos los labels, quitando la primera y la ultimas posiciones que es info sobrante
  chart.labels = [...pivot.data.table[0].value]
  chart.labels.shift()
  chart.labels.pop()

  // Agregamos el dataSet que es un array de objetos con formato {label:string, data: any[]}
  let datasetChart = [...pivot.data.table]
  datasetChart.shift()
  datasetChart.pop()

  datasetChart.forEach((element) => {
    for (let index = 0; index < element.value.length; index++) {
      if (element.value[index] == '') {
        element.value[index] = 0
      }
    }
    const item = {
      label: element.value[0],
      data: element.value,
      barThickness: 12,
      borderRadius: 5,
      backgroundColor: dameColorAleatorio()
    }
    item.data.shift()
    item.data.pop()
    chart.datasets.push(item)
  })
  chart.datasets[0].data.shift()
  chart.datasets.shift()
  console.log(chart)
  itemReactive.value = chart as any
}
</script>
<template>
  <h1>
    {{ props.type }}
  </h1>
  <Bar :data="itemReactive" :options="options" class="full" ref="chart" />
</template>
<style scoped>
.full {
  width: 550px;
  height: 247px;
}
</style>
