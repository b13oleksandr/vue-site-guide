<script>
import Vue from 'vue';
import VueSiteGuide from '@/vue-site-guide.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    VueSiteGuide
  }
});
</script>

<template>
  <div id="app">
    <vue-site-guide />
  </div>
</template>
