<template>
  <div>
    <button :disabled="!locationEnabled" style="margin-bottom: 10px">
      I am enabled when location is enabled
    </button>
    <div v-show="locationEnabled">
      latitude: {{ latitude }} longitude: {{ longitude }}
    </div>
  </div>
</template>

<script>
import { reactive, toRefs } from "vue";
export default {
  name: "App",
  setup() {
    const state = reactive({
      locationEnabled: false,
      latitude: undefined,
      longitude: undefined,
    });
    const checkForLocationPermissions = () => {
      if (!navigator.geolocation) {
        state.locationEnabled = false;
        return;
      }
      navigator.permissions
        .query({ name: "geolocation" })
        .then((result) => {
          if (result.state === "granted") {
            state.locationEnabled = true;
            navigator.geolocation.watchPosition((pos) => {
              console.log("position watcher");
              const { coords } = pos;
              state.longitude = coords.longitude;
              state.latitude = coords.latitude;
            });
          } else if (result.state === "denied") {
            state.locationEnabled = false;
          } else if (result.state === "prompt") {
            navigator.geolocation.getCurrentPosition(() => {});
          }
          result.onchange = () => {
            result.state === "granted"
              ? (state.locationEnabled = true)
              : (state.locationEnabled = false);
          };
        })
        .catch((error) => {
          console.log("Erro: ", error);
        });
    };

    checkForLocationPermissions();
    return { ...toRefs(state) };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
