<template lang="pug">
  v-flex(mb-5 v-if="existsAccount" v-bind:id="navTargetId")
    v-card
      v-card-title
        div.title VRF Key Link
      v-card-text
        v-radio-group(label="Link Action" v-model="linkAction" row)
          v-radio(
          v-for="pt in linkActions"
          :key="pt.type"
          :label="pt.label"
          :value="pt.type")
        v-text-field(
          label="Remote Account Public Key"
          v-model="linkedPublicKey"
          :counter="64"
          required)
        v-text-field(
          label="Max Fee"
          v-model="fee"
          min="0"
          type="number")
      v-card-actions
        v-btn(
        color="blue"
        class="white--text"
        @click="announceHandler") announce
      v-card-text
        tx-history(v-bind:history="history")
</template>

<script>
import { mapGetters } from 'vuex'
import { LinkAction, VrfKeyLinkTransaction, Deadline, UInt64, TransactionHttp } from 'symbol-sdk'
import TxHistory from '../history/TxHistory.vue'

export default {
  name: 'LinkVrfKey',
  components: {
    TxHistory
  },
  props: {
    navTargetId: {
      type: String,
      default () {
        return 'vrfLink'
      }
    }
  },
  data () {
    return {
      linkedPublicKey: '',
      linkAction: LinkAction.Link,
      linkActions: [
        { type: LinkAction.Link, label: 'Link' },
        { type: LinkAction.Unlink, label: 'Unlink' }
      ],
      fee: 0,
      history: []
    }
  },
  computed: {
    ...mapGetters('wallet', ['existsAccount', 'address', 'account', 'endpoint']),
    ...mapGetters('chain', ['generationHash']),
    ...mapGetters('env', ['feePlaceholder', 'publicKeyPlaceholder'])
  },
  mounted () {
    this.fee = this.feePlaceholder.default
    this.linkedPublicKey = this.publicKeyPlaceholder.alice
  },
  methods: {
    announceHandler (event) {
      const account = this.account
      const endpoint = this.endpoint
      const linkTransaction = VrfKeyLinkTransaction.create(
        Deadline.create(),
        this.linkedPublicKey,
        this.linkAction,
        account.address.networkType,
        UInt64.fromUint(this.fee)
      )
      const signedTx = account.sign(linkTransaction, this.generationHash)
      const txHttp = new TransactionHttp(endpoint)
      // eslint-disable-next-line no-console
      txHttp.announce(signedTx).toPromise().then(console.log)
      const historyData = {
        hash: signedTx.hash,
        apiStatusUrl: `${endpoint}/transactionStatus/${signedTx.hash}`
      }
      this.history.push(historyData)
    }
  }
}
</script>

<style scoped>

</style>
