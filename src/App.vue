<template>
  <div class="back">
    <Page class="app"></Page>
  </div>
</template>

<script>
import Page from "@/components/Page";
export default {
  name: 'App',
  components: {
    Page
  }
}
</script>

<style>
.back {

  height: 100vh;
  width: 100vw;

  padding: 10px;
  font-size: 20px;
  background-image: url("assets/sunset.jpg");
  background-size: cover;
}

.app {
  height: 97vh;
  width: 49%;
  background: rgba(38, 21, 76, 0.87);
  border-radius: 10px;
  padding: 10px;
}
</style>
