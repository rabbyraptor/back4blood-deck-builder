<template>
  <div id="app">
    <header>
      <h3>B4B Deck Builder</h3>
    </header>
    <DeckBuilder />
  </div>
</template>

<script>
import DeckBuilder from "./pages/deckBuilder/";

export default {
  name: "App",
  components: {
    DeckBuilder,
  },
};
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-size: 10px;
}

body {
  background: #000023;
  user-select: none;
}

header {
  margin: 4rem 0 1.9rem;
  text-align: center;
  color: white;
  font-size: 3rem;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
