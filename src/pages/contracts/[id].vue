<template>
  <Head>
    <Title>{{ APP_TITLE_SHORT }} | Smart Contract</Title>
  </Head>

  <page-header>
    Smart Contract

    <template v-if="!isContractFound">
      not found
    </template>

    <template #tooltip>
      {{ contractsHints.contract }}
      <app-link
        variant="primary"
        to="https://docs.aeternity.com/protocol/contracts/">
        Learn more
      </app-link>
    </template>
  </page-header>

  <loader-panel v-if="isLoading"/>

  <template v-if="isLoaded">
    <contract-details-panel
      class="contract-details__panel"
      :contract-details="contractDetails"/>

    <app-tabs>
      <app-tab title="Call Transactions">
        <contract-call-transactions-panel/>
      </app-tab>
      <app-tab title="Events">
        <contract-events-panel/>
      </app-tab>
    </app-tabs>
  </template>

  <not-found-panel v-if="hasError">
    Oops! We are sorry. The Smart Contract identified by
    <q>
      {{ route.params.id }}
    </q>
    was not found.
  </not-found-panel>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import ContractDetailsPanel from '@/components/ContractDetailsPanel'
import PageHeader from '@/components/PageHeader'
import { useContractDetailsStore } from '@/stores/contractDetails'
import NotFoundPanel from '@/components/NotFoundPanel'
import AppTabs from '@/components/AppTabs'
import AppTab from '@/components/AppTab'
import ContractEventsPanel from '@/components/ContractEventsPanel'
import { isDesktop } from '@/utils/screen'
import { contractsHints } from '@/utils/hints/contractsHints'
import ContractCallTransactionsPanel from '@/components/ContractCallTransactionsPanel'

const contractDetailsStore = useContractDetailsStore()
const { contractDetails } = storeToRefs(contractDetailsStore)
const { fetchContractDetails, fetchContractEvents } = contractDetailsStore
const route = useRoute()
const { isLoading } = useLoading()

await useAsyncData(() => fetchContractDetails(route.params.id))

const isContractFound = ref(true)
const hasError = computed(() => !isLoading.value && !isContractFound.value)
const isLoaded = computed(() => !isLoading.value && isContractFound.value)

const { error } = await useAsyncData(() => fetchContractDetails(route.params.id))

if (process.client && !error.value) {
  const limit = isDesktop() ? 10 : 3
  await useAsyncData(() => fetchContractEvents({
    queryParameters: `/v2/contracts/logs?contract_id=${route.params.id}&limit=${limit}`,
  }))
}

if (error.value) {
  isContractFound.value = false
  setResponseStatus(404, 'Smart Contract not found')
}
</script>

<style scoped>
.contract-details__panel {
  margin-bottom: var(--space-6);
}
</style>
