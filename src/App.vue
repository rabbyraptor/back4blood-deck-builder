<template>
  <div id="app">
    <header>
      <h3>B4B Deck Builder v1.0</h3>
      <a href="https://ko-fi.com/B0B36WCPF" target="_blank"
        ><img
          height="20"
          class="kofi-button"
          src="https://cdn.ko-fi.com/cdn/kofi2.png?v=3"
          border="0"
          alt="Buy Me a Coffee at ko-fi.com"
      /></a>
    </header>
    <router-view />
  </div>
</template>

<script>
export default {
  name: "App",
  mounted() {
    if (this.$route.fullPath != "/") {
      if (this.$route.query.deck) {
        this.$router.push({
          path: "/",
          query: { deck: this.$route.query.deck },
        });
      } else {
        this.$router.push("/");
      }
    }
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
  padding: 0 5rem;
  color: white;
  font-size: 1rem;
  z-index: 1;
  align-items: center;
  justify-content: space-between;
  background-color: #000;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.kofi-button {
  border: 0px;
  height: 2.5rem;
  margin-top: 0.4rem;
  margin-left: 2rem;
}
</style>
