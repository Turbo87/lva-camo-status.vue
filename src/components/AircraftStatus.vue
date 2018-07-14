<template>
  <div
          :title="error"
          class="aircraft-status"
          :class="classObject"
          @click="refresh">
  <span class="aircraft-status__callsign">{{callsign}}</span>
  <span class="aircraft-status__type">{{computedType}}</span>
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" class="aircraft-status__thumb-up"><path d="M17.62 10H20a2 2 0 0 1 2 2v8a2 2 0 0 1-2 2H8.5c-1.2 0-2.3-.72-2.74-1.79l-3.5-7-.03-.06A3 3 0 0 1 5 9h5V4c0-1.1.9-2 2-2h1.62l4 8zM16 11.24L12.38 4H12v7H5a1 1 0 0 0-.93 1.36l3.5 7.02a1 1 0 0 0 .93.62H16v-8.76zm2 .76v8h2v-8h-2z"></path></svg>
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" class="aircraft-status__thumb-down"><path d="M6.38 14H4a2 2 0 0 1-2-2V4c0-1.1.9-2 2-2h11.5c1.2 0 2.3.72 2.74 1.79l3.5 7 .03.06A3 3 0 0 1 19 15h-5v5a2 2 0 0 1-2 2h-1.62l-4-8zM8 12.76L11.62 20H12v-7h7c.13 0 .25-.02.38-.08a1 1 0 0 0 .55-1.28l-3.5-7.02A1 1 0 0 0 15.5 4H8v8.76zM6 12V4H4v8h2z"></path></svg>
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" class="aircraft-status__error"><path d="M12 22a10 10 0 1 1 0-20 10 10 0 0 1 0 20zm0-2a8 8 0 1 0 0-16 8 8 0 0 0 0 16zM10.59 8.59a1 1 0 1 1-1.42-1.42 4 4 0 1 1 5.66 5.66l-2.12 2.12a1 1 0 1 1-1.42-1.42l2.12-2.12A2 2 0 0 0 10.6 8.6zM12 18a1 1 0 1 1 0-2 1 1 0 0 1 0 2z"></path></svg>
  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" class="aircraft-status__refresh"><path class="heroicon-ui" d="M6 18.7V21a1 1 0 0 1-2 0v-5a1 1 0 0 1 1-1h5a1 1 0 1 1 0 2H7.1A7 7 0 0 0 19 12a1 1 0 1 1 2 0 9 9 0 0 1-15 6.7zM18 5.3V3a1 1 0 0 1 2 0v5a1 1 0 0 1-1 1h-5a1 1 0 0 1 0-2h2.9A7 7 0 0 0 5 12a1 1 0 1 1-2 0 9 9 0 0 1 15-6.7z"></path></svg>
  </div>
</template>

<script>
import VueTimers from "vue-timers/mixin";
import axios from "axios";

export default {
  mixins: [VueTimers],

  name: "AircraftStatus",

  props: {
    id: String,
    callsign: String,
    type: String
  },

  data() {
    return {
      isLoading: false,
      error: null,
      status: null
    };
  },

  computed: {
    computedType() {
      let status = this.status;

      if (this.type) {
        return this.type;
      } else if (!status) {
        return null;
      } else if (status["easa-type"] === status["easa-variant"]) {
        return status["easa-type"];
      } else {
        return `${status["easa-type"]} ${status["easa-variant"]}`;
      }
    },

    isError() {
      return Boolean(this.error);
    },

    isAirworthy() {
      if (!this.status) {
        return null;
      } else {
        return (
          this.status.camo === "airworthy" &&
          (this.status.ato === "unknown" || this.status.ato === "airworthy")
        );
      }
    },

    classObject() {
      return {
        "aircraft-status--loading": this.isLoading,
        "aircraft-status--error": this.isError,
        "aircraft-status--ok": this.isAirworthy === true,
        "aircraft-status--nope": this.isAirworthy === false
      };
    }
  },

  timers: {
    update: { time: 60000, autostart: true, repeat: true, immediate: true }
  },

  methods: {
    async update() {
      if (this.isLoading) return;

      try {
        this.isLoading = true;
        this.error = null;
        this.status = null;

        let response = await axios.get(
          `https://api.camo-europe.aero/statuses/${this.id}`
        );
        this.status = response.data.data.attributes;
      } catch (error) {
        this.error = error;
      } finally {
        this.isLoading = false;
      }
    },

    refresh() {
      this.$timer.stop("update");
      this.$timer.start("update");
    }
  }
};
</script>

<style scoped lang="scss">
.aircraft-status {
  position: relative;

  max-width: 450px;
  margin: 8px;
  padding: 16px;

  background-color: white;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.18);
  cursor: pointer;

  transition: background-color 0.2s, color 0.2s;
}

.aircraft-status__refresh,
.aircraft-status__error,
.aircraft-status__thumb-up,
.aircraft-status__thumb-down {
  position: absolute;

  right: 16px;
  top: 16px;
  width: 52px;
  height: 52px;

  opacity: 0;
  transition: opacity 0.2s;
}

.aircraft-status--loading {
  background-color: #eee;
  color: #999;
  fill: #999;

  .aircraft-status__refresh {
    opacity: 1;
  }
}

.aircraft-status--error {
  background: hsla(42, 100%, 50%, 1);
  color: rgba(0, 0, 0, 0.7);
  fill: rgba(0, 0, 0, 0.7);

  .aircraft-status__error {
    opacity: 1;
  }
}

.aircraft-status--ok {
  background: hsla(120, 100%, 35%, 1);
  color: rgba(255, 255, 255, 0.7);
  fill: rgba(255, 255, 255, 0.7);

  .aircraft-status__thumb-up {
    opacity: 1;
  }
}

.aircraft-status--nope {
  background: hsla(0, 100%, 35%, 1);
  color: rgba(255, 255, 255, 0.7);
  fill: rgba(255, 255, 255, 0.7);

  .aircraft-status__thumb-down {
    opacity: 1;
  }
}

.aircraft-status__callsign {
  font-size: 40px;
  line-height: 52px;
  margin: 0 8px;
}

.aircraft-status__type {
  font-size: 16px;
  line-height: 16px;
}

@media screen and (max-width: 400px) {
  .aircraft-status__callsign {
    font-size: 40px;
    line-height: 40px;
  }

  .aircraft-status__type {
    display: block;
    margin: 8px 8px 0;
  }

  .aircraft-status__refresh,
  .aircraft-status__error,
  .aircraft-status__thumb-up,
  .aircraft-status__thumb-down {
    right: 22px;
    top: 22px;
  }
}
</style>
