<template>
  <div>
    <StrategyInPlay strategy_mode="my-alerts" />
  </div>
</template>

<script>
import StrategyInPlay from "~/components/sections/StrategyInPlay";
export default {
  head() {
    return {
      title: this.$titles.in_play_strategies,
    };
  },
  components: {
    StrategyInPlay,
  },
};
</script>

<style></style>
