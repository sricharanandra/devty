<script setup lang="ts">
import Input from '@/components/InputComponent.vue'
import InputOptions from '@/components/InputOptionsComponent.vue'
import { ref } from 'vue'
import axios from 'axios'
import { UiCard } from 'balm-ui'

const types = ['A', 'AAAA', 'CNAME', 'MX', 'TXT']
const selectedType = ref('')
const results = ref(null)
const value = ref('')

async function performLookup() {
  try {
    const response = await axios.get(
      `https://cloudflare-dns.com/dns-query?type=${selectedType.value}`,
      {
        headers: {
          Accept: 'application/dns-json'
        }
      }
    )
    results.value = response.data
  } catch (error) {
    console.error(error)
  }
}

function clearInput() {
  value.value = ''
}

function handleUpdate(newValue: string) {
  value.value = newValue
  performLookup()
}

let timer: ReturnType<typeof setTimeout>

function useThrottles(value: string, func: Function, timeout: number = 100) {
  clearTimeout(timer)

  timer = setTimeout(() => {
    func(value)
  }, timeout)
}
</script>

<template>
  <div class="dns-lookup">
    <div class="inputSection">
      <InputOptions @reset="clearInput" @paste="handleUpdate" :copyContent="value" />
      <Input
        input-type="textarea"
        placeholder="Enter DNS to lookup"
        class="no-resize inputText"
        label="Input"
        v-model="value"
        @input="handleUpdate(value)"
        @keyup.enter="performLookup()"
      ></Input>
      <button @click="performLookup">Lookup</button>
    </div>
    <ui-card class="result-card" v-if="results" title="DNS Lookup Results">
      <template #content>
        <ul>
          <li v-for="(value, key) in results" :key="key">
            <strong>{{ key }}:</strong> {{ value }}
          </li>
        </ul>
      </template>
    </ui-card>
  </div>
</template>

<style scoped>
.container {
  width: 100%;
  padding: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
}

.inputSection,
.outputSection {
  color: var(--color-text);
  display: flex;
  flex-direction: column;
}

.inputSection .inputText {
  flex-grow: 1;
}

.inputSection {
  margin-left: 20px;
}
.outputSection {
  margin-right: 5px;
}

.outputSection .output {
  flex-grow: 1;
  border-radius: 5px;
  overflow: hidden;
}

@media screen and (max-width: 800px) {
  .container {
    display: block;
  }
}
</style>
