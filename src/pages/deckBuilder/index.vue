<template>
  <div class="deck-builder" v-if="!this.loading">
    <div class="search-wrapper">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Start typing to search. Press Enter to select input."
        ref="search"
      />
    </div>

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
        <div class="custom-deck-buttons">
          <button class="clear-deck-button" @click="clearCustomDeck()">
            Reset
          </button>
          <button class="copy-deck-button" @click="copyCustomDeck()">
            {{ this.copyMessage }}
          </button>
          <button class="copy-deck-button" @click="randomizeDeck()">
            Randomize
          </button>
        </div>
      </div>

      <h4 class="custom-deck__title">
        Custom Deck: {{ this.customDeck.length }} / 15
      </h4>

      <draggable
        class="custom-deck__grid"
        :group="{ name: 'cards', put: true, pull: true }"
        :list="this.customDeck"
        @start="drag = true"
        @end="drag = false"
        @change="
          pushDeckToRouter();
          resetCopyMessage();
        "
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
          v-for="(card, index) in this.filteredSearchList"
          :key="index + index"
          @click.prevent="addCardToCustomCards(card)"
        >
          <img
            class="card-image"
            :src="card.image"
            :alt="card.title"
            loading="lazy"
          />
          <div class="card-content">
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
      <div
        v-if="this.filteredSearchList.length < 1"
        class="empty-search-result"
      >
        No matches found.
      </div>
      <p class="disclaimer">
        Notice of Non-Affiliation and Disclaimer: We are not affiliated,
        associated, authorized, endorsed by, or in any way officially connected
        with Back 4 Blood, or any of its subsidiaries or its affiliates. The
        official Back 4 Blood website can be found at
        <a href="https://back4blood.com/" target="_blank"
          >https://back4blood.com/</a
        >. The Back 4 Blood, Turtle Rock Studios, Warner Bros. as well as
        related names, marks, emblems and images are registered trademarks of
        their respective owners. <br /><br />Thank you for using b4bdeck.com!
      </p>
    </div>

    <!-- CUSTOM DECK DATA  -->
    <CustomDeckData :data="customDeckData"></CustomDeckData>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import CustomDeckData from "@/components/customDeckData/customDeckData.vue";

export default {
  data() {
    return {
      drag: false,
      customDeck: [],
      cardData: {},
      searchQuery: null,
      loading: true,
      cardDataApi: "/cardData.json",
      calculatedEffects: [],
      isMinimized: false,
      importedDeck: [],
      cardsToImport: [],
      copyMessage: "Copy to clipboard",
    };
  },
  components: {
    draggable,
    CustomDeckData,
  },
  computed: {
    // Returns a filtered list of cards, based on the search query.
    filteredSearchList() {
      let searchQuery = this.searchQuery;

      function findTitle(card) {
        return card.title.toLowerCase().includes(searchQuery.toLowerCase());
      }

      function findType(card) {
        return card.effects.filter((item) => {
          return item.type.toLowerCase().includes(searchQuery.toLowerCase());
        }).length;
      }

      function findAmount(card) {
        for (let i = 0; i < card.effects.length; i++)
          if (card.effects[i].amount && !card.effects[i].isPercentage) {
            return card.effects[i].amount.toString().includes(searchQuery);
          } else if (card.effects[i].amount && card.effects[i].isPercentage) {
            let percentageString = card.effects[i].amount.toString() + "%";
            return percentageString.includes(searchQuery);
          }
      }

      if (this.searchQuery) {
        let searchResults = this.cardData.filter((card) => {
          return findTitle(card) || findType(card) || findAmount(card);
        });
        return searchResults;
      } else return this.cardData;
    },

    // Groups single-string effects and calculable effects
    customDeckData() {
      let customDeck = this.customDeck;
      let mainEffects = [];
      let secondaryEffects = [];

      for (var i = 0; i < customDeck.length; i++) {
        for (var l = 0; l < customDeck[i].effects.length; l++) {
          if (customDeck[i].effects[l].amount != null) {
            // If the effect has an amount it is calculable: It is a main effect.
            mainEffects.push(customDeck[i].effects[l]);
          } else {
            // Else it is NOT calulable: It is a secondary effect.
            secondaryEffects.push(customDeck[i].effects[l]);
          }
        }
      }

      // Find matches in effect types and group them
      let groupedEffects = (xs, key) => {
        return xs.reduce((rv, x) => {
          (rv[x[key]] = rv[x[key]] || []).push(x);
          return rv;
        }, {});
      };

      return {
        mainEffects: this.calculateValues(groupedEffects(mainEffects, "type")),
        secondaryEffects: secondaryEffects,
      };
    },
  },
  methods: {
    addCardToCustomCards(card) {
      if (this.cardData.find((item) => item == card)) {
        if (
          this.isCustomDeckNotFull() == true &&
          !this.cardExistsInCustomDeck(card)
        ) {
          this.customDeck.push(card);
          this.pushDeckToRouter();
          this.resetCopyMessage();
        } else if (this.cardExistsInCustomDeck(card)) {
          this.customDeck.splice(this.customDeck.indexOf(card), 1);
          this.pushDeckToRouter();
          this.resetCopyMessage();
        }
      }
    },
    pushDeckToRouter() {
      let routeIds = "";

      this.customDeck.forEach((card) => {
        if (this.customDeck.indexOf(card) == 0) {
          routeIds += card.id.toString();
        } else {
          routeIds += "," + card.id.toString();
        }
      });

      this.$router.push({ query: { deck: routeIds } });
    },
    removeCardFromCustomCards(card) {
      this.customDeck.splice(this.customDeck.indexOf(card), 1);
      this.pushDeckToRouter();
      this.resetCopyMessage();
    },
    cardExistsInCustomDeck(card) {
      var customDeck = this.customDeck;
      for (var i = 0; i < customDeck.length; i++) {
        if (customDeck[i].id === card.id) {
          return true;
        }
      }
    },
    isCustomDeckNotFull() {
      if (this.customDeck.length < 15) {
        return true;
      } else {
        return false;
      }
    },
    clearCustomDeck() {
      this.customDeck = [];
      this.$router.push("/");
      this.resetCopyMessage();
    },
    copyCustomDeck() {
      navigator.clipboard.writeText(window.location.href);
      this.copyMessage = "Deck copied!";
    },
    getRandomInt(max) {
      return Math.floor(Math.random() * max);
    },
    randomizeDeck() {
      const deckSize = 15;

      this.customDeck = [];

      const cardNumbers = new Set();
      while(cardNumbers.size !== deckSize) {
        cardNumbers.add(this.getRandomInt(this.cardData.length));
      }

      cardNumbers.forEach((cardNumber) => {
        const card = this.cardData[cardNumber];
        this.addCardToCustomCards(card);
      });
    },
    resetCopyMessage() {
      this.copyMessage = "Copy to clipboard";
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
          id: i + 1,
        });
      }
      this.cardData = transformedCardData;
    },
    calculateValues(mainEffects) {
      let groupedEffects = mainEffects;
      let calculatedMainEffects = [];
      /* 
      let p = (percentages) => {
        return percentages / 100;
      }; */

      for (const entry in groupedEffects) {
        let numbers = 0;
        let percentages = 0;
        let typeName = "";

        if (groupedEffects[entry].length > 0) {
          for (const effect in groupedEffects[entry]) {
            typeName = groupedEffects[entry][effect].type;

            if (groupedEffects[entry][effect].isPercentage) {
              percentages += groupedEffects[entry][effect].amount;
            } else {
              numbers += groupedEffects[entry][effect].amount;
            }
          }

          if (numbers != 0 || percentages != 0) {
            if (typeName == "Health") {
              if (numbers === 0) {
                calculatedMainEffects.push({
                  type: typeName,
                  totalAmount: this.isPositiveNumber(percentages) + "%",
                });
              } else if (percentages === 0) {
                calculatedMainEffects.push({
                  type: typeName,
                  totalAmount: this.isPositiveNumber(numbers),
                });
              } else {
                calculatedMainEffects.push({
                  type: typeName,
                  totalAmount:
                    this.isPositiveNumber(percentages) +
                    "% (" +
                    this.isPositiveNumber(numbers) +
                    ")",
                });
              }
            }
            // If type is not "Health"
            else if (numbers === 0) {
              calculatedMainEffects.push({
                type: typeName,
                totalAmount: this.isPositiveNumber(percentages) + "%",
              });
            } else {
              calculatedMainEffects.push({
                type: typeName,
                totalAmount: this.isPositiveNumber(
                  numbers
                ) /* * (1 + p(percentages))*/,
              });
            }
          }
        }
      }

      return calculatedMainEffects;
    },
    getBackgroundImage(image) {
      if (image) return 'background-image: url("' + image + '")';
      else return null;
    },
    minimize() {
      this.isMinimized = !this.isMinimized;
    },
    isPositiveNumber(n) {
      if (n > 0) {
        return "+" + n;
      } else return n;
    },
    importDeck() {
      let deckParams = this.$route.query.deck;

      if (deckParams) {
        let cardsToImport = deckParams;
        cardsToImport = cardsToImport.split(",").map(Number);
        this.cardsToImport = cardsToImport;
        let arr = [];

        for (let i = 0; i < cardsToImport.length; i++) {
          this.addCardToCustomCards(
            this.cardData.find((card) => card.id == cardsToImport[i])
          );
          if (arr.length >= 15) {
            break;
          }
        }

        this.importedDeck = arr;
      }
    },
  },
  async beforeMount() {
    // Fetch data from api
    const data = await fetch(this.cardDataApi);
    this.cardData = await data.json();
    this.addDataToCards();
    this.loading = false;
    this.importDeck();
  },
  created() {
    window.addEventListener("keydown", (e) => {
      if (/^[a-z0-9]$/i.test(e.key)) {
        // If keyboard input is a letter or a number, focus the input field.
        this.$refs.search.focus();
      }
    });
  },
};
</script>

<style src="./index.scss" lang="scss" scoped></style>
