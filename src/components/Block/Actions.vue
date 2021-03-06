<template>
  <Block v-if="plugins.length > 0 && ts >= payload.end" :title="'Actions'">
    <UiButton
      v-for="plugin in plugins"
      :key="plugin"
      @click="execute(plugin)"
      :loading="loading"
      :disabled="!$auth.isAuthenticated"
      class="width-full button--submit"
    >
      Submit on-chain
    </UiButton>
  </Block>
</template>

<script>
import { mapActions } from 'vuex';
import plugins from '@snapshot-labs/snapshot.js/src/plugins';

export default {
  props: ['id', 'space', 'payload', 'results'],
  data() {
    return {
      loading: false
    };
  },
  computed: {
    ts() {
      return (Date.now() / 1e3).toFixed();
    },
    winningChoice() {
      let winningChoice = 0;
      let winningScore = 0;
      this.results.totalScores.forEach((score, i) => {
        if (score[0] > winningScore) {
          winningChoice = i + 1;
          winningScore = score[0];
        }
      });
      return winningChoice;
    },
    plugins() {
      if (this.space && this.space.plugins && this.space.plugins['aragon'])
        return ['aragon'];
      return [];
    }
  },
  methods: {
    ...mapActions(['notify']),
    async execute(plugin) {
      this.loading = true;
      const action = new plugins[plugin]();
      try {
        const tx = await action.action(
          this.web3.network.key,
          this.$auth.web3,
          this.space.plugins[plugin],
          this.payload.metadata.plugins[plugin],
          this.id,
          this.winningChoice
        );
        const receipt = await tx.wait();
        console.log('Receipt', receipt);
        this.notify(['green', `You did it, congrats!`]);
      } catch (e) {
        console.error(e);
      }
      this.loading = false;
    }
  }
};
</script>
