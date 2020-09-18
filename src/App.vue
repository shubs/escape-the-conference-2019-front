<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<style>

body {
  font-family: 'Quicksand', sans-serif;
  color: #111;;
}

.bg-grey {
  background-color: #111;
}
.font-grey {
  color: #111;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

:root {
  --cell-size: 2fr;
  --edge-size: 1fr;
}
</style>
