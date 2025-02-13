<template>
  <v-card v-if="instance.instance?.status != 'open'" @click="startConnect" elevation="0">
    <v-alert
      icon="mdi-qrcode-scan"
      color="deep-orange-darken-2"
      class="text-white"
    >
      <div class="d-flex justify-space-between align-center flex-wrap">
        <b>Telefone não conectado</b>
        <v-btn @click="startConnect" size="small"> Conectar </v-btn>
      </div>
    </v-alert>
  </v-card>
  <v-dialog v-model="dialog" max-width="350px">
    <v-card :loading="loading && qrCode">
      <v-card-text>
        <v-img v-if="qrCode" :src="qrCode" width="300px" height="300px" />
        <v-card
          v-else
          width="300px"
          height="300px"
          variant="outlined"
          elevation="0"
        >
          <v-card-text class="d-flex justify-center align-center h-100">
            <v-progress-circular v-if="loading" indeterminate color="primary" />
            <v-icon v-else-if="error" size="x-large">mdi-qrcode-remove</v-icon>
          </v-card-text>
        </v-card>
        <v-btn
          text
          size="small"
          block
          @click="loadQr"
          :loading="loading"
          :disabled="disabledRefresh"
          variant="tonal"
          class="mt-2"
        >
          <v-icon start size="small">mdi-refresh</v-icon>
          Atualizar
        </v-btn>

        <v-alert type="error" v-if="error" class="mt-2">
          {{ Array.isArray(error) ? error.join(", ") : error }}
        </v-alert>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text @click="dialog = false" :disabled="loading"> Cancel </v-btn>
        <v-spacer></v-spacer>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { useAppStore } from "@/store/app";
import instanceController from "@/services/instanceController";

export default {
  name: "SettingsModal",
  data: () => ({
    dialog: false,
    error: false,

    loading: false,
    qrCode: null,
    success: false,

    timeout: null,
    disabledRefresh: false,

    AppStore: useAppStore(),
  }),
  methods: {
    async loadQr() {
      try {
        this.loading = true;
        this.error = false;
        clearTimeout(this.timeout);

        const response = await instanceController.connect(
          this.instance.instance.instanceName
        );

        if (response.base64) this.qrCode = response.base64;
        else if (response.instance) {
          this.dialog = false;
          this.AppStore.reconnect();
          return;
        } else
          throw new Error(
            "Não foi possível carregar o QR Code, se o erro persistir, reinicie a API e tente novamente."
          );

        this.timeout = setTimeout(this.loadQr, 40000);
        this.disabledRefresh = true;
        setTimeout(() => (this.disabledRefresh = false), 5000);
      } catch (e) {
        this.error = e.message?.message || e.message || e;
      } finally {
        this.loading = false;
      }
    },
    async startConnect() {
      clearTimeout(this.timeout);
      this.dialog = true;
      this.error = false;

      await this.loadQr();
      await this.AppStore.reconnect();
    },
  },
  props: {
    instance: {
      type: Object,
      required: true,
    },
  },

  emits: ["close"],
};
</script>
