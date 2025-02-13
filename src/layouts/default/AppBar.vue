<template>
  <v-app-bar flat>
    <v-app-bar-title class="flex-shrink-0">
      <v-btn variant="text" @click="$router.push({ name: 'instances' })">
        <v-img src="@/assets/logo.png" height="24" width="24" class="mr-2" />
        Evolution Manager
      </v-btn>
    </v-app-bar-title>
    <v-icon v-if="AppStore.connecting" color="info">
      mdi-loading mdi-spin
    </v-icon>
    <v-chip
      v-else-if="AppStore.validConnection"
      color="success"
      style="max-width: 35vw"
    >
      <v-icon color="success" start> mdi-check-circle </v-icon>
      {{
        AppStore.connection.host.replace(/https?:\/\//, "").replace(/\/$/, "")
      }}
    </v-chip>
    <v-icon v-else color="error"> mdi-alert-circle </v-icon>
    <v-btn @click="openSettings" icon>
      <v-icon>mdi-cog</v-icon>
    </v-btn>
    <v-btn @click="toggleTheme" icon>
      <v-icon>mdi-{{ dark ? "white-balance-sunny" : "weather-night" }}</v-icon>
    </v-btn>
  </v-app-bar>
  <SettingsModal ref="settings" />
</template>

<script>
import SettingsModal from "@/components/modal/Settings.vue";
import { useAppStore } from "@/store/app";
import { useTheme } from "vuetify";

export default {
  name: "AppBar",
  data: () => ({
    AppStore: useAppStore(),
    theme: useTheme(),
  }),
  components: {
    SettingsModal,
  },
  methods: {
    toggleTheme() {
      const theme = this.theme.global.current.dark ? "light" : "dark";
      this.theme.global.name = theme;
      localStorage.setItem("theme", theme);
    },
    openSettings() {
      this.$refs.settings.open();
    },
    async loadConnectionFromUrl() {
      try {
        const { connection } = this.$route.query;
        if (!connection) return null;
        this.$router.replace({ query: {} });

        const json = atob(connection);
        const data = JSON.parse(json);
        if (!data.host || !data.globalApiKey) return null;

        await this.AppStore.setConnection(data);
        return data;
      } catch (e) {
        console.error(e);
        return null;
      }
    },
  },
  computed: {
    dark() {
      return this.theme.global.current.dark;
    },
  },
  async mounted() {
    const urlConnection = await this.loadConnectionFromUrl();
    if (!urlConnection) await this.AppStore.loadConnection();
    if (!this.AppStore.validConnection) this.openSettings();
  },
};
</script>
