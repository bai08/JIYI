<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <HelloWorld msg="Welcome to Your Vue.js App" style="height: 100%" />
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/model.vue';

export default {
  name: 'HomeView',
  components: {
    HelloWorld
  }
};
</script>
<style scoped>
.home {
  height: 100%;
  /* height: 100%; */
}
</style>
