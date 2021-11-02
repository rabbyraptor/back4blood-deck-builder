<template>
  <div class="custom-deck-data">
    <h3 class="custom-deck-data__title">Main effects</h3>
    <div class="main-data">
      <div
        v-for="(effect, index) in data.effectsArray"
        :key="effect.type + index"
        class="effect data-grid"
      >
        <span style="font-weight: bold">
          {{ effect.type }}
        </span>
        <span>
          {{ effect.totalAmount }}
        </span>
      </div>
      <span
        class="data-grid effect--placeholder"
        v-if="data.effectsArray.length < 1"
      >
        None - add cards to see effects.
      </span>
    </div>

    <h3 class="custom-deck-data__title">Secondary effects</h3>
    <div class="secondary-data">
      <div
        v-for="(effect, index) in data.effectArray"
        :key="index"
        class="effect data-grid"
      >
        <span>
          {{ effect.type }}
        </span>
      </div>
      <span
        class="data-grid effect--placeholder"
        v-if="data.effectsArray.length < 1"
      >
        None - add cards to see effects.
      </span>
    </div>
  </div>
</template>

<script>
export default {
  props: ["data"],
};
</script>

<style src="./customDeckData.scss" scoped lang="scss"></style>
