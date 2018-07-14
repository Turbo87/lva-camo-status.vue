<template>
  <div class="app">
    <h1 class="title">CAMO Status LVA</h1>

    <AircraftStatus id="2UHDjFVtA" callsign="D-0980" type="Discus CS"/>
    <AircraftStatus id="WTsHJRdZ" callsign="D-8784" type="ASK 21"/>
    <AircraftStatus id="37JFc7WUm" callsign="D-9041" type="Duo Discus"/>
    <AircraftStatus id="26Gk3e81q" callsign="D-EFAC" type="DR 400/180R"/>
  </div>
</template>

<script>
import AircraftStatus from "./components/AircraftStatus.vue";

export default {
  name: "app",
  components: {
    AircraftStatus
  }
};
</script>

<style lang="scss">
body {
  background-color: white;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica,
    Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
}

.title {
  margin: 24px 32px;
  font-size: 24px;
}
</style>
