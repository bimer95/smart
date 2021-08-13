<template>
  <div class="smart">
    <div class="smart__body">
      <div class="smart__head">
        <b-button class="smart__btn-connect"
                  @click="handleConnectWallet"
        >
          Connect Wallet
        </b-button>
      </div>

      <label for="amount">Amount</label>
      <div class="smart__amount">
        <input
            class="smart__input"
            id="amount"
            v-model="amountInput"
            placeholder="amount"
        >

        <b-dropdown
            v-model="currencyValue"
            :text="currencies[currencyValue]"
        >
          <template v-for="(item,i) in currencies">
            <b-dropdown-item
                :key="i"
                @click="currencyValue = i;"
            >
              {{ item }}
            </b-dropdown-item>
          </template>
        </b-dropdown>
      </div>

      <label for="recipient">Address (recipient)</label>
      <input
          class="smart__input"
          id="recipient"
          v-model="recipientInput"
          placeholder="recipient"
          @input="changeAllowance"
      >
      <span>Balance: {{ balance }} {{ currencies[currencyValue] }}</span>
      <span>Allowance:{{ allowance }}</span>
      <div class="smart__btns">
        <b-button
            @click="approve"
            class="smart__btn"
        >
          Approve
        </b-button>
        <b-button
            class="smart__btn"
            @click="transfer"
        >
          Transfer
        </b-button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'App',
  data() {
    return {
      amountInput: '',
      recipientInput: '',
      currencyValue: 0,
      currencies: [],
      symbol: '',
    };
  },
  computed: {
    ...mapGetters({
      isConnected: 'web3/getIsConnected',
      tokens: 'web3/getTokens',
      userAddress: 'web3/getUserAddress',
      balance: 'web3/getBalance',
      allowance: 'web3/getAllowance',
    }),
  },
  watch: {
    async currencyValue() {
      this.symbol = this.currencies[this.currencyValue];
      await this.$store.dispatch('web3/setCurrentBalance', this.symbol);
      const recipientAddress = this.recipientInput;
      await this.$store.dispatch('web3/setCurrentAllowance', { symbol: this.symbol, recipientAddress });
    },
  },
  mounted() {
    this.symbol = this.currencies.length ? this.currencies[this.currencyValue] : '';
  },
  methods: {
    async handleConnectWallet() {
      this.currencies = [];
      await this.$store.dispatch('web3/connectWallet');
      if (this.isConnected) {
        await this.$store.dispatch('web3/setToken', process.env.DLS);
        await this.$store.dispatch('web3/setToken', process.env.CFI);
        await this.$store.dispatch('web3/setToken', process.env.LP);
        await this.$store.dispatch('web3/setToken', process.env.USDT);

        Object.keys(this.tokens).forEach((key) => {
          this.currencies.push(key);
        });
        this.symbol = this.currencies[this.currencyValue];
        await this.$store.dispatch('web3/setCurrentBalance', this.symbol);
        await this.$store.dispatch('web3/setCurrentAllowance', { symbol: this.symbol, recipientAddress: this.recipientInput });
      }
    },
    async changeAllowance() {
      if (!this.isConnected) return;
      const recipientAddress = this.recipientInput;
      await this.$store.dispatch('web3/setCurrentAllowance', { symbol: this.symbol, recipientAddress });
    },
    async approve() {
      const recipientAddress = this.recipientInput;
      const amount = Number(this.amountInput);
      await this.$store.dispatch('web3/approve', { symbol: this.symbol, recipientAddress, amount });
    },
    async transfer() {
      const recipientAddress = this.recipientInput;
      const amount = Number(this.amountInput);
      await this.$store.dispatch('web3/transfer', { symbol: this.symbol, recipientAddress, amount });
    },
  },
};
</script>

<style lang="scss" scoped>
.smart {
  display: flex;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  &__body {
  }
  &__head {
    display: flex;
    justify-content: flex-end;
  }
  &__btn {
    background-color: #63BCD8;
    width: 100%;
    max-width: 200px;
  }
  &__btn-connect {
    display: flex;
    background-color: #63BCD8;
  }
  &__btns {
    display: flex;
    justify-content: center;
    gap: 20px;
  }
  &__amount {
    display: flex;
    gap: 10px;
  }
  &__body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    //align-items: center;
    position: relative;
    padding: 20px;

    max-width: 700px;
  }
  &__input {
    background-color: #F3F5FA;
    width: 600px;
    height: 40px;
    border-radius: 10px;
    border-color: white;
  }
  &__drop {
    background-color: #63BCD8;
  }
}
</style>
