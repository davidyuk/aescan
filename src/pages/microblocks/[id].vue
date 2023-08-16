<template>
  <Head>
    <Title>{{ APP_TITLE_SHORT }} | Microblock</Title>
  </Head>

  <page-header>
    Microblock

    <template v-if="!isMicroblockFound">
      not found
    </template>

    <template #tooltip>
      {{ microblocksHints.microblock }}
    </template>
  </page-header>

  <loader-panel v-if="isLoading"/>

  <template v-if="isLoaded">
    <microblock-details-panel :microblock-details="microblockDetails"/>
    <microblock-transactions-panel/>
  </template>

  <not-found-panel v-if="hasError">
    Oops! We are sorry. The microblock identified by
    <q>
      {{ route.params.id }}
    </q>
    was not found.
  </not-found-panel>
</template>

<script setup>
import { storeToRefs } from 'pinia'
import { useRoute } from 'nuxt/app'

import { microblocksHints } from '@/utils/hints/microblocksHints'
import { useMicroblockDetailsStore } from '@/stores/microblockDetails'
import PageHeader from '@/components/PageHeader'
import NotFoundPanel from '@/components/NotFoundPanel'
import MicroblockDetailsPanel from '@/components/MicroblockDetailsPanel'
import MicroblockTransactionsPanel from '~/components/MicroblockTransactionsPanel'

const route = useRoute()
const { isLoading } = useLoading()

const microblockDetailsStore = useMicroblockDetailsStore()
const { microblockDetails } = storeToRefs(microblockDetailsStore)
const { fetchMicroblock } = microblockDetailsStore

const isMicroblockFound = ref(true)
const isLoaded = computed(() => !isLoading.value && isMicroblockFound.value)
const hasError = computed(() => !isLoading.value && !isMicroblockFound.value)

const { error } = await useAsyncData(async() => {
  await fetchMicroblock(route.params.id)
  return true
})

if (error.value) {
  isMicroblockFound.value = false
  setResponseStatus(404, 'Microblock not found')
}
</script>
