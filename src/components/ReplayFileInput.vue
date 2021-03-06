<template>
  <b-row>
    <b-col lg="7">
      <b-form @submit="onSubmit($event)">
        <b-alert variant="danger" dismissible :show="hasError">
          {{ errorMessage }}
        </b-alert>
        <fieldset :disabled="isInputDisabled">
          <b-form-group label="Replay file" label-for="input-replay-file">
            <b-form-file
              id="input-replay-file"
              v-model="file"
              accept=".orarep"
            ></b-form-file>
            <small class="form-text text-muted">
              Choose or drag and drop an <strong>.orarep</strong> file.
            </small>
          </b-form-group>
          <Submit />
        </fieldset>
      </b-form>
    </b-col>
  </b-row>
</template>

<script>
import { EventBus } from '../event-bus.js';
import Utils from '../utils.js';

import Submit from './Submit.vue';

export default {
  components: {
    Submit,
  },
  data() {
    return {
      hasError: false,
      errorMessage: null,
      file: null,
      replayJSON: null,
    };
  },
  computed: {
    isInputDisabled() {
      return this.$store.state.settings.isInputDisabled;
    },
  },
  methods: {
    onSubmit(e) {
      e.preventDefault();

      if (this.isInputDisabled) {
        return;
      }

      this.clearError();

      if (!this.file) {
        return;
      }

      this.$store.commit('settings/setLoadingState', true);

      const formData = new FormData(e.target);
      formData.append('data', this.file);

      setTimeout(() => {
        fetch(process.env.VUE_APP_OPENRA_API_ENDPOINT, {
          method: 'POST',
          body: formData
        }).then(
          response => response.json()
        ).then(replayJSON => {
          this.replayJSON = Utils.cleanUpBuild(replayJSON);
          this.replayDataReady();
        }).catch(e => {
          this.setError(e);
          this.$store.commit('settings/setLoadingState', false);
        });
      }, 0);
    },
    replayDataReady() {
      EventBus.$emit('ReplayDataReady', this.replayJSON);
      this.$storageManager.storeReplay(this.replayJSON);
    },
    setError(e) {
      this.hasError = true;
      this.errorMessage = `${e.name}: ${e.message}`;
    },
    clearError() {
      this.hasError = false;
      this.errorMessage = null;
    },
  },
};
</script>
