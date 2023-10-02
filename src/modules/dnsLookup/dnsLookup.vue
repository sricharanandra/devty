<script setup lang="ts">
import Input from '@/components/InputComponent.vue'
import InputOptions from '@/components/InputOptionsComponent.vue'
import useThrottle from '@/hooks/useThrottle'
import { ref } from 'vue'
import axios from 'axios'
import { UiSelect, UiButton, UiCard } from 'balm-ui'

const types = ['A', 'AAAA', 'CNAME', 'MX', 'TXT']
const selectedType = ref('')
const results = ref(null)

function clearInput() {
  value.value = ''
}
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
</script>

<template>
  <div class="dns-lookup">
    <balm-card class="result-card" v-if="results" title="DNS Lookup Results">
      <template #content>
        <ul>
          <li v-for="(value, key) in results" :key="key">
            <strong>{{ key }}:</strong> {{ value }}
          </li>
        </ul>
      </template>
    </balm-card>
    <div class="inputSection">
      <InputOptions @reset="clearInput" @paste="handleUpdate" :copyContent="value" />
      <Input
        input-type="textarea"
        placeholder="Enter dns to lookup"
        class="no-resize inputText"
        label="Input"
        :value="value"
        @update:value="useThrottle($event, handleUpdate)"
      ></Input>
    </div>
  </div>
</template>

<style scoped>
.container {
  width: 100%;
  padding: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
}

.inputSection {
  color: var(--color-text);
  display: flex;
  flex-direction: column;
}

.inputSection,
.outputSection {
  margin-right: 5px;
  margin-left: 20px;
}

.hash {
  width: 100%;
  margin: 7.5px 0;
}

.inputSection .inputText {
  flex-grow: 1;
}

@media screen and (max-width: 800px) {
  .container {
    display: block;
  }
}
</style>
