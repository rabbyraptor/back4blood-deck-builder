<template>
  <div class="custom-deck-data" :class="{ 'is-minimized': isMinimized }">
    <div class="main-data">
      <h3 class="custom-deck-data__title">Main effects</h3>
      <div
        v-for="(effect, index) in data.mainEffects"
        :key="effect.type + index"
        class="effect data-grid"
      >
        <span style="font-weight: bold">
          {{ effect.type }}
        </span>
        <span
          :class="{ 'negative-number': isNegativeNumber(effect.totalAmount) }"
        >
          {{ effect.totalAmount }}
        </span>
      </div>
      <span
        class="data-grid effect--placeholder"
        v-if="data.mainEffects.length < 1"
      >
        None - add cards to see effects.
      </span>
    </div>

    <div class="secondary-data">
      <h3 class="custom-deck-data__title">Secondary effects</h3>
      <div
        v-for="(effect, index) in data.secondaryEffects"
        :key="index"
        class="effect data-grid"
      >
        <span>
          {{ effect.type }}
        </span>
      </div>
      <span
        class="data-grid effect--placeholder"
        v-if="data.mainEffects.length < 1"
      >
        None - add cards to see effects.
      </span>
    </div>
    <button
      v-if="!isMinimized"
      class="minimize-deck-data-button"
      @click="minimize()"
    >
      -
    </button>
    <button
      v-if="isMinimized"
      class="minimize-deck-data-button"
      @click="
        isMinimized = !isMinimized;
        $emit('minimize');
      "
    >
      +
    </button>
  </div>
</template>

<script>
export default {
  props: ["data"],
  data() {
    return {
      isMinimized: false,
    };
  },
  methods: {
    isNegativeNumber(n) {
      if (n.charAt(0) == "-") {
        console.log("Is minus");
        return true;
      }
    },
    minimize() {
      this.isMinimized = !this.isMinimized;
      this.$emit("minimize");
    },
  },
};
</script>

<style src="./customDeckData.scss" scoped lang="scss"></style>
