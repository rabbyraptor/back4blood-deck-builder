<template>
  <div class="deck-builder" v-if="!this.loading">
    <div class="custom-deck">
      <div class="custom-deck__grid skeleton">
        <div class="card">1</div>
        <div class="card">2</div>
        <div class="card">3</div>
        <div class="card">4</div>
        <div class="card">5</div>
        <div class="card">6</div>
        <div class="card">7</div>
        <div class="card">8</div>
        <div class="card">9</div>
        <div class="card">10</div>
        <div class="card">11</div>
        <div class="card">12</div>
        <div class="card">13</div>
        <div class="card">14</div>
        <div class="card">15</div>
      </div>

      <h4 class="custom-deck__title">
        Custom Deck 1: {{ this.customDeck.length }}/15
      </h4>

      <draggable
        class="custom-deck__grid"
        :group="{ name: 'cards', put: true, pull: true }"
        :list="this.customDeck"
        @start="drag = true"
        @end="drag = false"
        :animation="200"
      >
        <transition-group
          type="transition"
          :name="!drag ? 'flip-list' : null"
          class="custom-deck__grid"
        >
          <div
            class="card"
            v-for="card in this.customDeck"
            :key="card.id"
            :style="getBackgroundImage(card.image)"
          ></div>
        </transition-group>
      </draggable>
    </div>

    <!-- DRAGGABLE DIVIDER  -->

    <div class="card-collection">
      <draggable
        class="card-collection__grid"
        :list="this.transformedCardData"
        :group="{ name: 'cards', pull: !isCustomDeckFull(), put: true }"
        :sort="true"
      >
        <div
          class="card"
          :class="{
            'dragging-is-disabled': cardExistsInCustomDeck(card.id),
          }"
          v-for="card in this.transformedCardData"
          :key="card.id"
          :style="getBackgroundImage(card.image)"
        >
          <h4 class="card__title card__effect">
            {{ card.title }}
          </h4>
          <p class="card__type card__effect">Type: {{ card.type }}</p>
          <p class="card__effect">
            {{ card.effect }}
          </p>
          <p
            class="card__effect"
            v-for="(effect, index) in card.effects"
            :key="effect.type + index"
          >
            {{ effect.type }}
          </p>
        </div>
      </draggable>
    </div>

    <!-- CUSTOM DECK DATA  -->
    <CustomDeckData :data="calculatedValues"></CustomDeckData>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import CustomDeckData from "@/components/customDeckData/customDeckData.vue";

export default {
  data() {
    return {
      drag: false,
      disabled: false,
      customDeck: [],
      transformedCardData: [],
      cardData: {},
      cardImages: {},
      localImages: {},
      loading: true,
      imagePath: "/images/",
      oldApi:
        "https://opensheet.vercel.app/1zURAO8DELx_EN1D8YkuJ8hw_WQ2jY1mcLE4D8ElMYUo/B4B%20Card%20List%20-%20Master",
      cardDataApi: "/franksCards.json",
      calculatedEffects: [],
    };
  },
  components: {
    draggable,
    CustomDeckData,
  },
  computed: {
    customDeckEffects() {
      let customDeck = this.customDeck;
      let effectsArray = [];
      let effectArray = [];

      for (var i = 0; i < customDeck.length; i++) {
        for (var l = 0; l < customDeck[i].effects.length; l++) {
          if (customDeck[i].effects[l].amount != null) {
            effectsArray.push(customDeck[i].effects[l]);
          } else {
            effectArray.push(customDeck[i].effects[l]);
          }
        }
      }

      let groupedEffects = function(xs, key) {
        return xs.reduce(function(rv, x) {
          (rv[x[key]] = rv[x[key]] || []).push(x);
          return rv;
        }, {});
      };

      return {
        effectsArray: groupedEffects(effectsArray, "type"),
        effectArray: effectArray,
      };
    },

    addedEffectValuesComp() {
      return this.addedEffectValues();
    },
    calculatedValues() {
      let types = this.customDeckEffects.effectsArray;

      let objectToReturn = [];
      function p(percentages) {
        return percentages / 100;
      }

      for (const type in types) {
        let numbers = 0;
        let percentages = 0;
        let typeName = "";

        if (types[type].length > 0) {
          for (const effect in types[type]) {
            typeName = types[type][effect].type;

            if (types[type][effect].isPercentage) {
              percentages += types[type][effect].amount;
            } else {
              numbers += types[type][effect].amount;
            }
          }

          if (numbers != 0 || percentages != 0) {
            if (numbers === 0) {
              objectToReturn.push({
                type: typeName,
                totalAmount: percentages + "%",
              });
            } else {
              objectToReturn.push({
                type: typeName,
                totalAmount: numbers * (1 + p(percentages)),
              });
            }
          }
        }
      }

      return {
        effectsArray: objectToReturn,
        effectArray: this.customDeckEffects.effectArray,
      };
    },
  },
  methods: {
    cardExistsInCustomDeck(card) {
      var customDeck = this.customDeck;
      for (var i = 0; i < customDeck.length; i++) {
        if (customDeck[i].id === card) {
          return true;
        }
      }
    },
    isCustomDeckFull() {
      if (this.customDeck.length < 15) {
        return false;
      } else {
        return true;
      }
    },
    addDataToCards() {
      let numberOfCards = this.cardData.length;
      let transformedCardData = [];

      for (let i = 0; i < numberOfCards; i++) {
        transformedCardData.push({
          type: this.cardData[i].type,
          title: this.cardData[i].title,
          image: this.cardData[i].image,
          effects: this.cardData[i].effects,
          id: i,
        });
      }
      this.transformedCardData = transformedCardData;
    },
    getBackgroundImage(image) {
      if (image) return 'background-image: url("' + image + '")';
      else return null;
    },
  },
  async beforeMount() {
    // Fetch data from api
    const data = await fetch(this.cardDataApi);
    this.cardData = await data.json();
    this.addDataToCards();
    this.loading = false;
  },
};
</script>

<style src="./index.scss" lang="scss" scoped></style>
