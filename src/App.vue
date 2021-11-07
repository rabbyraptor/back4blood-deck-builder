<template>
  <div id="app">
    <header>
      <h3>B4B Deck Builder</h3>
    </header>
    <router-view />
  </div>
</template>

<script>
export default {
  name: "App",
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
  background: #000;
  background-image: url("/images/b4b_titlescreen_bg.jpg");
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  user-select: none;
  background-attachment: fixed;
}

header {
  display: flex;
  position: fixed;
  width: 100%;
  height: 5rem;
  margin: 0;
  padding-right: 6rem;
  color: white;
  font-size: 1rem;
  z-index: 1;
  align-items: center;
  justify-content: flex-end;
  background-color: #000;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
