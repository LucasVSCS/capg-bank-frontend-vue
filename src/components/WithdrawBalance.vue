<template>
  <v-sheet rounded="lg" min-height="40vh">
    <p class="text-h6 text-center text-uppercase font-weight-black pt-4">
      <v-icon large color="black darken-2">mdi-cash-fast</v-icon>
      Sacar saldo
    </p>
    <v-divider />
    <v-row align="center" justify="center" class="pt-12">
      <v-col cols="12" sm="9">
        <validation-observer ref="observer">
          <v-form @submit.prevent="submit">
            <validation-provider
              v-slot="{ errors }"
              name="withdrawAmount"
              rules="required|double|greaterThanZero"
            >
              <v-text-field-money
                v-model="withdrawAmount"
                label="Digite o valor do saque"
                name="withdrawAmount"
                :error-messages="errors"
                v-bind:properties="{
                  prefix: 'R$',
                  readonly: false,
                  disabled: false,
                  outlined: true,
                  clearable: true,
                  placeholder: ' ',
                }"
                v-bind:options="{
                  locale: 'pt-BR',
                  length: 10,
                  precision: 2,
                  empty: null,
                }"
                required
              />
            </validation-provider>
            <v-btn
              type="submit"
              class="rounded-0"
              color="#000000"
              x-large
              block
              dark
              >Realizar saque</v-btn
            >
          </v-form>
        </validation-observer>
      </v-col>
    </v-row>
  </v-sheet>
</template>


<script>
import instance from "@/services/api.js";
import Cookie from "js-cookie";

export default {
  data: () => ({
    withdrawAmount: null,
  }),
  computed: {
    params() {
      return {
        withdrawAmount: this.withdrawAmount,
      };
    },
  },
  methods: {
    async submit() {
      const valid = await this.$refs.observer.validate();

      if (valid) {
        instance
          .post("account/withdraw-balance", JSON.stringify(this.params), {
            headers: {
              Authorization: "Bearer " + Cookie.get("_capg-bank_token"),
            },
          })
          .then((response) => {
            if (response.data.newBalance >= 0) {
              this.$toasted.success(response.data.message);
              this.$emit("updateBalance", response.data.newBalance);
              this.$emit(
                "updateTransactionsHistory",
                response.data.transactionData
              );
              this.clear();
            } else {
              this.$toasted.error(response.data.message);
            }
          })
          .catch((error) => {
            this.$toasted.error(error.response.data.message);
          });
      }
    },
    clear() {
      this.withdrawAmount = 0;
      this.$refs.observer.reset();
    },
  },
};
</script>
