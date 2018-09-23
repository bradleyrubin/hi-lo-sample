<template>
  <div style="text-align:left">
    <h5>Scores</h5>
    <p
      v-for="(player, index) in players"
      :key="player.name">
      {{ player.name }}: {{ player.points }}
      <span v-if="index === currentPlayerIndex">*</span>
    </p>
  </div>
</template>

<script>

export default {
  components: {
  },
  props: {
    players: {
      type: Array,
      required: true,
    },
    currentPlayerIndex: {
      type: Number,
      required: true,
    },
  },
  computed: {
    sampleComputed() {
      return 1;
    },
  },

};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
