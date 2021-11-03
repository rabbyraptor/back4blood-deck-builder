<template>
  <div class="deck-builder" v-if="!this.loading">
    <div class="custom-deck">
      <div class="custom-deck__grid skeleton">
        <div class="card">1 &nbsp; Empty</div>
        <div class="card">2 &nbsp; Empty</div>
        <div class="card">3 &nbsp; Empty</div>
        <div class="card">4 &nbsp; Empty</div>
        <div class="card">5 &nbsp; Empty</div>
        <div class="card">6 &nbsp; Empty</div>
        <div class="card">7 &nbsp; Empty</div>
        <div class="card">8 &nbsp; Empty</div>
        <div class="card">9 &nbsp; Empty</div>
        <div class="card">10 &nbsp; Empty</div>
        <div class="card">11 &nbsp; Empty</div>
        <div class="card">12 &nbsp; Empty</div>
        <div class="card">13 &nbsp; Empty</div>
        <div class="card">14 &nbsp; Empty</div>
        <div class="card">15 &nbsp; Empty</div>
        <button class="clear-button" @click="clearCustomDeck()">
          Reset
        </button>
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
        ghost-class="ghost"
      >
        <transition-group
          type="transition"
          :name="!drag ? 'flip-list' : null"
          class="custom-deck__grid"
        >
          <div
            class="card"
            v-for="(card, index) in this.customDeck"
            :key="card.id"
            :style="getBackgroundImage(card.image)"
            @click.prevent="removeCardFromCustomCards(card)"
          >
            <div class="card__fading-overlay">
              <span>
                <span class="card__index">{{ index + 1 }}</span
                ><span class="card__title">{{ card.title }}</span>
              </span>
            </div>
          </div>
        </transition-group>
      </draggable>
    </div>

    <!-- DRAGGABLE DIVIDER  -->

    <div class="card-collection">
      <div class="card-collection__grid">
        <div
          class="card"
          :class="{
            'dragging-is-disabled': cardExistsInCustomDeck(card),
          }"
          v-for="(card, index) in this.transformedCardData"
          :key="index + index"
          :style="getBackgroundImage(card.image)"
          @click.prevent="addCardToCustomCards(card)"
        >
          <h4 class="card__title card__effect">
            {{ card.title }}
          </h4>
          <p class="card__type card__effect card__drag-hidden-effect">
            Type: {{ card.type }}
          </p>
          <p class="card__effect card__drag-hidden-effect">
            {{ card.effect }}
          </p>
          <p
            class="card__effect card__drag-hidden-effect"
            v-for="(effect, index) in card.effects"
            :key="index + index"
          >
            {{ effect.type }}
          </p>
        </div>
      </div>
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
    addCardToCustomCards(card) {
      if (
        this.isCustomDeckFull() == true &&
        !this.cardExistsInCustomDeck(card)
      ) {
        this.customDeck.push(card);
      } else if (this.cardExistsInCustomDeck(card)) {
        this.customDeck.splice(this.customDeck.indexOf(card), 1);
      }
    },
    removeCardFromCustomCards(card) {
      this.customDeck.splice(this.customDeck.indexOf(card), 1);
    },
    cardExistsInCustomDeck(card) {
      var customDeck = this.customDeck;
      for (var i = 0; i < customDeck.length; i++) {
        if (customDeck[i].id === card.id) {
          return true;
        }
      }
    },
    isCustomDeckFull() {
      if (this.customDeck.length < 15) {
        return true;
      } else {
        return false;
      }
    },
    clearCustomDeck() {
      this.customDeck = [];
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
    getCustomCardBackgroundImage(image) {
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
