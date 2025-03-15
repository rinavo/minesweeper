<script setup>

import MineSweeper from "@/components/MineSweeper.vue";
import {ref} from "vue";

const w = ref(5)
const h = ref(5)
const r = ref(0.1)
const f = ref(false)
</script>

<template>
  <main class="flex justify-center items-center w-full ">
    <div class="mx-10 max-w-40 mr-10">
      <fieldset class="fieldset">
        <legend class="fieldset-legend">Width</legend>
        <input type="text" v-model="w" class="input" placeholder="Type here" />
      </fieldset>
      <fieldset class="fieldset">
        <legend class="fieldset-legend">Height</legend>
        <input type="text" v-model="h" class="input" placeholder="Type here" />
      </fieldset>
      <fieldset class="fieldset">
        <legend class="fieldset-legend">Mine gen rate</legend>
        <input type="text" v-model="r" class="input" placeholder="Type here" />
      </fieldset>

      <button class="mt-4 w-40 btn btn-primary" @click="f=!f">Reset</button>
    </div>

    <MineSweeper :height="h" :width="w" :generate-rate="r" :key="h-w-r-f"/>
  </main>
</template>

<style scoped>
</style>
