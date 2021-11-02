<template>
  <div class="custom-deck-data">
    <h3 class="custom-deck-data__subtitle">Effects:</h3>
    <div
      v-for="(effect, index) in data.effectsArray"
      :key="effect.type + index"
      class="effect"
    >
      <p style="font-weight: bold">
        {{ effect.type }}
      </p>
      <span>
        {{ effect.totalAmount }}
      </span>
    </div>

    <h3 class="custom-deck-data__subtitle">Other effects:</h3>
    <div
      v-for="(effect, index) in data.effectArray"
      :key="index"
      class="effect"
    >
      <p style="font-weight: bold">
        {{ effect.type }}
      </p>
    </div>
  </div>
</template>

<script>
export default {
  props: ["data"],
};
</script>

<style src="./customDeckData.scss" scoped lang="scss"></style>
