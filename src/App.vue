<script>
import Navbar from "./components/partials/Navbar.vue";
import Footer from "./components/partials/Footer.vue";


export default {
  components: {
    Navbar, Footer
  },
};
</script>

<template>
  <div id="app">
    <Navbar />
    <router-view />
    <Footer />
  </div>
</template>

<style>
</style>