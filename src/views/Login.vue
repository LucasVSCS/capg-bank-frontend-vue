<template>
  <v-app>
    <v-main>
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center" dense>
          <v-col cols="12" sm="8" md="4" lg="4">
            <v-card elevation="0">
              <v-img
                src="@/assets/logo.png"
                alt="Capg-bank logo"
                contain
                height="200"
              ></v-img>
              <v-card-text>
                <validation-observer ref="observer">
                  <v-form @submit.prevent="submit">
                    <validation-provider
                      v-slot="{ errors }"
                      name="Email"
                      rules="required|email"
                    >
                      <v-text-field
                        v-model="email"
                        :error-messages="errors"
                        label="Digite seu e-mail"
                        name="email"
                        prepend-inner-icon="mdi-email"
                        type="email"
                        class="rounded-0"
                        outlined
                        required
                      ></v-text-field>
                    </validation-provider>
                    <validation-provider
                      v-slot="{ errors }"
                      name="Senha"
                      rules="required"
                    >
                      <v-text-field
                        v-model="password"
                        :error-messages="errors"
                        label="Digite sua senha"
                        name="password"
                        prepend-inner-icon="mdi-lock"
                        type="password"
                        class="rounded-0"
                        outlined
                        required
                      ></v-text-field>
                    </validation-provider>
                    <v-btn
                      type="submit"
                      class="rounded-0"
                      color="#000000"
                      x-large
                      block
                      dark
                      >Login</v-btn
                    >
                  </v-form>
                </validation-observer>
              </v-card-text>
              <v-card-actions class="ml-6 mr-6 text-center">
                <p>
                  Ao continuar, você concorda com a<span
                    class="pl-2 text-decoration-underline"
                    >Política</span
                  >
                  e os<span class="pl-2 text-decoration-underline"
                    >Termos de Uso</span
                  >
                  da Capg-bank
                </p>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import instance from "@/services/api.js";
import Cookie from "js-cookie";

export default {
  beforeCreate() {
    if (Cookie.get("_capg-bank_token")) {
      this.$router.push("/");
    }
  },
  data: () => ({
    email: "",
    password: null,
  }),
  computed: {
    params() {
      return {
        email: this.email,
        password: this.password,
      };
    },
  },
  methods: {
    async submit() {
      const valid = await this.$refs.observer.validate();

      if (valid) {
        instance
          .post("auth/login", JSON.stringify(this.params))
          .then((response) => {
            if (response.data.access_token) {
              this.setupCookie(response.data.access_token);
              setTimeout(() => {
                this.$router.push("/");
              }, 500);
            } else {
              this.$toasted.error("Credenciais inválidas");
            }
          });
      }
    },
    setupCookie(accessToken) {
      Cookie.set("_capg-bank_token", accessToken);
      this.$toasted.success("Usuário autenticado com sucesso");
      this.$refs.observer.reset();
    },
  },
};
</script>
