<template>
  <div class="deck-builder" v-if="!this.loading">
    <div class="deck-grid">
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

    <h4 class="deck-grid__title">
      Custom Deck 1: {{ this.customDeck.length }}/15
    </h4>
    <draggable
      class="deck-grid"
      :group="{ name: 'cards', put: true, pull: true }"
      :list="this.customDeck"
      @start="drag = true"
      @end="drag = false"
      :animation="200"
    >
      <transition-group
        type="transition"
        :name="!drag ? 'flip-list' : null"
        class="deck-grid"
      >
        <div
          class="card"
          v-for="card in this.customDeck"
          :key="card.id"
          :style="getBackgroundImage(card.image)"
        ></div>
      </transition-group>
    </draggable>

    <!-- DRAGGABLE DIVIDER  -->

    <div>
      <draggable
        class="card-grid"
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
    <div class="custom-deck-data">
      <h3 class="custom-deck-data__subtitle">Effects:</h3>
      <div
        v-for="(effect, index) in calculatedValues.effectsArray"
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
        v-for="(effect, index) in calculatedValues.effectArray"
        :key="index"
        class="effect"
      >
        <p style="font-weight: bold">
          {{ effect.type }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import draggable from "vuedraggable";
//import lodash from "lodash";

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

      /* function getTeamStatus(type) {
        console.log(type.id);
        for (const effect in types[type]) {
          console.log(types[type][effect].appliesToTeam);
        }
        return true;
      } */

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

          //let skod = getTeamStatus(types[type]);

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
  updated() {},
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.deck-grid {
  display: grid;
  grid-template-rows: repeat(15, 45px);
  align-items: baseline;
  gap: 6px;
  position: fixed;
  left: 5rem;
  top: 5rem;
  height: 787px;
  min-width: 190px;
}

.deck-grid__title {
  color: white;
  position: fixed;
  top: 3.5rem;
  left: 5rem;
  width: max-content;
}

.ghost,
.deck-grid .card {
  background-color: #222;
  height: 45px;
  width: 190px;
  color: white;
  background-repeat: no-repeat;
  background-size: cover;
}

.card-grid {
  display: grid;
  width: max-content;
  margin: auto;
  justify-content: center;
  grid-template-columns: repeat(4, auto);
  gap: 2rem;
}

.card-wrapper {
}

.dragging-is-disabled {
  opacity: 0.5;
  pointer-events: none;
}

.card-grid .card {
  height: 325px;
  width: 215px;
  color: white;
  padding: 10% 7.5%;
  border: 1px solid crimson;
  border-radius: 5px;
  display: grid;
  grid-auto-rows: max-content;
  position: relative;
  background-repeat: no-repeat;
  background-position: 50% 50%;
  background-size: contain;
  background-color: crimson;
}

.card__title {
  margin-bottom: 0.6rem;
}

.card__type {
  margin-bottom: 4rem;
}
.card__effect {
  position: relative;
  color: transparent;
}

.card__title,
.card__type,
.card__effect {
  opacity: 1;
}

.custom-deck-data {
  position: fixed;
  top: 5rem;
  right: 5rem;
  background: white;
  width: 300px;
  padding: 2rem;
  font-size: 0.9rem;
  border-radius: 2px;
  overflow-y: scroll;
  max-height: calc(100vh - 10rem);
}

.custom-deck-data__title {
  margin-bottom: 1rem;
}

.custom-deck-data__subtitle:not(:first-of-type) {
  margin-top: 2rem;
}

.custom-deck-data .effect {
  font-size: 0.8rem;
  padding: 1rem 0;
  border-bottom: 1px solid #eee;
}

.custom-deck-data .effect:last-of-type {
  margin-bottom: 0;
}

.list-move {
  transition: transform 0.5s;
}
</style>
