<script setup lang="ts">

const isLoading = ref(false)

const fetchData = async () => {
  await new Promise(r => setTimeout(r, 2000));
}

onMounted(async () => {
  isLoading.value = true
  await fetchData();
  isLoading.value = false
})

</script>

<template>
  <div>
    <!-- <div i-carbon-campsite text-4xl inline-block /> -->
    <h2 text-xl font-bold>
      Chief Med
    </h2>
    <p text-lg> Today </p>
    <div v-if="!isLoading">
      Loaded
    </div>
    <div v-if="isLoading">Loading...</div>

  </div>
</template>
