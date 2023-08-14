<template>
  <table>
    <tr>
      <th>
        Call transaction
        <hint-tooltip>
          {{ contractsHints.eventsCallTransaction }}
        </hint-tooltip>
      </th>
      <th>
        Created
        <hint-tooltip>
          {{ contractsHints.eventsCreated }}
        </hint-tooltip>
      </th>
      <th>
        Name
        <hint-tooltip>
          {{ contractsHints.eventsName }}
        </hint-tooltip>
      </th>
      <th>
        Data
        <hint-tooltip>
          {{ contractsHints.eventsData }}
        </hint-tooltip>
      </th>
    </tr>
    <tr
      v-for="(event, eventIndex) in contractEvents.data"
      :key="event.callTxHash">
      <td class="contract-events-table__transaction">
        <value-hash-ellipsed
          :hash="event.callTxHash"
          :link-to="`/transactions/${event.callTxHash}`"/>
      </td>
      <td>
        <div>
          <app-link
            :to="`/keyblocks/${event.createdHeight}`">
            {{ event.createdHeight }}
          </app-link>
        </div>
        <datetime-label :datetime="event.created"/>
      </td>
      <td>
        {{ event.eventName ? event.eventName : 'N/A' }}
      </td>
      <td>
        <pre>
          {{ event }}
        </pre>
        After decoding:
        <pre>
          {{ decodedEvents[eventIndex] ? stringifyDecodedEvent(decodedEvents[eventIndex]) : 'Decoding event...' }}
        </pre>
      </td>
    </tr>
  </table>
</template>

<script setup>
import { Node, AeSdk, CompilerHttp } from '@aeternity/aepp-sdk'
import { contractsHints } from '@/utils/hints/contractsHints'
import DatetimeLabel from '@/components/DatetimeLabel'
import CopyChip from '@/components/CopyChip'
import ValueHashEllipsed from '@/components/ValueHashEllipsed'
import { formatEllipseHash } from '@/utils/format'

const removeLineBreaks = str => {
  return str.toString().replaceAll('\n', '')
}

const props = defineProps({
  contractEvents: {
    type: Object,
    required: true,
  },
})

const decodedEvents = ref([])

// example contract deployed to testned
const address = 'ct_2NoQGtBcFCHi8ZcCUJtBFekvN4jscYLQxAXuPXuiBDUZrxG4hF'
const aci = [
  {
    namespace: {
      name: 'ListInternal',
      typedefs: [],
    },
  },
  {
    namespace: {
      name: 'List',
      typedefs: [],
    },
  },
  {
    namespace: {
      name: 'String',
      typedefs: [],
    },
  },
  {
    contract: {
      event: {
        variant: [
          {
            Transfer: [
              'address',
              'address',
              'int',
            ],
          },
          {
            Mint: [
              'address',
              'int',
            ],
          },
        ],
      },
      functions: [
        {
          arguments: [],
          name: 'aex9_extensions',
          payable: false,
          returns: {
            list: [
              'string',
            ],
          },
          stateful: false,
        },
        {
          arguments: [
            {
              name: 'name',
              type: 'string',
            },
            {
              name: 'decimals',
              type: 'int',
            },
            {
              name: 'symbol',
              type: 'string',
            },
            {
              name: 'initial_owner_balance',
              type: {
                option: [
                  'int',
                ],
              },
            },
          ],
          name: 'init',
          payable: false,
          returns: 'FungibleToken.state',
          stateful: false,
        },
        {
          arguments: [],
          name: 'meta_info',
          payable: false,
          returns: 'FungibleToken.meta_info',
          stateful: false,
        },
        {
          arguments: [],
          name: 'total_supply',
          payable: false,
          returns: 'int',
          stateful: false,
        },
        {
          arguments: [],
          name: 'owner',
          payable: false,
          returns: 'address',
          stateful: false,
        },
        {
          arguments: [],
          name: 'balances',
          payable: false,
          returns: 'FungibleToken.balances',
          stateful: false,
        },
        {
          arguments: [
            {
              name: 'account',
              type: 'address',
            },
          ],
          name: 'balance',
          payable: false,
          returns: {
            option: [
              'int',
            ],
          },
          stateful: false,
        },
        {
          arguments: [
            {
              name: 'to_account',
              type: 'address',
            },
            {
              name: 'value',
              type: 'int',
            },
          ],
          name: 'transfer',
          payable: false,
          returns: {
            tuple: [],
          },
          stateful: true,
        },
      ],
      kind: 'contract_main',
      name: 'FungibleToken',
      payable: false,
      state: {
        record: [
          {
            name: 'owner',
            type: 'address',
          },
          {
            name: 'total_supply',
            type: 'int',
          },
          {
            name: 'balances',
            type: 'FungibleToken.balances',
          },
          {
            name: 'meta_info',
            type: 'FungibleToken.meta_info',
          },
        ],
      },
      typedefs: [
        {
          name: 'meta_info',
          typedef: {
            record: [
              {
                name: 'name',
                type: 'string',
              },
              {
                name: 'symbol',
                type: 'string',
              },
              {
                name: 'decimals',
                type: 'int',
              },
            ],
          },
          vars: [],
        },
        {
          name: 'balances',
          typedef: {
            map: [
              'address',
              'int',
            ],
          },
          vars: [],
        },
      ],
    },
  },
]

watch(() => props.contractEvents, async() => {
  console.log('events:', props.contractEvents.data)
  decodedEvents.value = await Promise.all(props.contractEvents.data.map(decodeEvent))
}, { immediate: true })

async function decodeEvent(event, index) {
  const node = new Node('https://testnet.aeternity.io') // ideally host your own node
  const aeSdk = new AeSdk({
    nodes: [{ name: 'testnet', instance: node }],
    onCompiler: new CompilerHttp('https://v7.compiler.aepps.com'), // ideally host your own compiler
  })

  const contract = await aeSdk.initializeContract({ aci, address })

  /* ### decode event data by transaction hash ### */
  const txHash = event.callTxHash
  // aeSdk is an instance of the AeSdk class
  const txInfo = await aeSdk.api.getTransactionInfoByHash(txHash)
  // decode events using contract instance
  const decodedUsingContract = contract.$decodeEvents(txInfo.callInfo.log)

  /* ### decode event data by providing raw event data ###
  const decodedUsingContract = contract.$decodeEvents([{
    address,
    data: 'DATA GOES HERE',
    topics: [
      // TOPICS GO HERE
    ]
  }]) */

  console.log(decodedUsingContract)
  return decodedUsingContract
}

function stringifyDecodedEvent(decodedEvent) {
  return JSON.stringify(decodedEvent, (key, value) =>
    typeof value === 'bigint'
      ? value.toString()
      : value, // return everything else unchanged
  )
}
</script>
<style scoped>
.contract-events-table {
  &__transaction {
    white-space: nowrap;
  }

  &__event-data {
    word-wrap: anywhere;
    max-width: 450px;
  }
}
</style>
