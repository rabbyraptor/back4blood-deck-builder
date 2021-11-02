<template>
  <div id="app">
    <header>
      <h2>B4B Deck Builder</h2>
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

<style>
body {
  background: #000023;
}
header {
  margin: 4rem 0 4rem;
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
