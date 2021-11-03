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
  height: 100%;
  width: 100%;
  display: flex;
}

body {
  display: flex;
  height: 100%;
  width: 100%;
  background: #000023;
  background-image: url("/images/b4b_titlescreen_bg.jpg");
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  user-select: none;
  background-attachment: fixed;
}

header {
  margin: 0;
  padding-right: 6rem;
  height: 5rem;
  color: white;
  font-size: 1rem;
  position: absolute;
  display: flex;
  flex-grow: 1;
  width: 100%;
  align-items: center;
  justify-content: flex-end;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
