<script setup lang="ts">
import axios from 'axios'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'
import { Ref } from 'vue'
dayjs.extend(utc)
dayjs.extend(timezone)

const isLoading = ref(false)
const r3shift: Ref<any> = ref([[]])

const fetchData = async () => {
  // Sleep
  // await new Promise(r => setTimeout(r, 2000));

  // const sheetId = import.meta.env.VITE_SHEET_R3
  // const sheetRange = import.meta.env.VITE_RANGE_R3

  let sheetId = ''
  let sheetRange = ''
  // const medconfigUrl = 'https://gcp.limzz.xyz/med-config/get-config-all?table=chiefmed'
  const medconfigUrl = 'http://35.199.151.231/med-config/get-config-all?table=chiefmed'

  // Get Chief med config from med-config-backend
  try {
    const confRes = (await axios.get(medconfigUrl)).data
    sheetId = confRes[0].value.split('spreadsheets/')[1].split('/')[1]
    sheetRange = confRes[1].value
  } catch (err) {
    console.error(err)
  }

  if (sheetId === '') {
    sheetId = import.meta.env.VITE_SHEET_R3
    sheetRange = import.meta.env.VITE_RANGE_R3
  }

  const apiKey = import.meta.env.VITE_API_KEY
  let url = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/${sheetRange}`
  let urlMain = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/A1:E50`

  try {
    const res = await axios.get(urlMain, {
      params: {
        key: apiKey
      }
    })
    // console.log(res.data.values)
    r3shift.value = res.data.values.filter((e:any) => e[0]==today.add(543, 'y').format('D/M/YY'))
    if (r3shift.value.length == 0) {
      r3shift.value = res.data.values.filter((e:any) => e[0]==today.format('M/D/YY'))
    }
    if (r3shift.value.length == 0) {
      const res2 = await axios.get(url, {
        params: {
          key: apiKey
        }
      })
      r3shift.value = res2.data.values.filter((e:any) => e[0]==today.add(543, 'y').format('D/M/YY'))
      if (r3shift.value.length == 0) {
        r3shift.value = res2.data.values.filter((e:any) => e[0]==today.format('M/D/YY'))
      }
    }
  } catch (err) {
    console.error(err)
    r3shift.value = [['','error','error','error']]
  }
}

let today = dayjs().tz("Asia/Bangkok")
if (today.hour() < 8) today = today.subtract(1, 'd')

onMounted(async () => {
  isLoading.value = true
  await fetchData();
  isLoading.value = false
})

</script>

<template>
  <div>
    <h2 text-3xl font-bold>
      <div i-carbon-reminder-medical text-2xl inline-block></div>
      Chief Med
    </h2>
    <p text-lg p-4> {{ today.format('dddd DD/MM/YYYY') }} </p>
    <hr/>
    <div v-if="!isLoading">
      <div p-3>
        <h2 text-lg font-bold>เวรบน</h2>
        <span>{{ r3shift[0][1] }}</span>
      </div>
      <hr/>
      <div p-3>
        <h2 text-lg font-bold>เวรกลาง</h2>
        <span>{{ r3shift[0][2] }}</span>
      </div>
      <hr/>
      <div p-3>
        <h2 text-lg font-bold>เวรล่าง</h2>
        <span>{{ r3shift[0][3] }}</span>
      </div>
      
    </div>
    <div v-if="isLoading">Loading...</div>

  </div>
</template>
