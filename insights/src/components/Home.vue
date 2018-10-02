<template>
  <div id="home">
    <div id="master-search-container">
      <search
        placeholder="placeholder"
        :items="poslanci"
      ></search>
    </div>
    <visualisation
      :selectedMPs="selectedMPs"
    ></visualisation>
    <words :poslanci="poslanci"></words>
  </div>
</template>
<script>
import Visualisation from './Visualisation';
import Search from './Search';
import Words from './Words';
import * as poslanci from '../assets/poslanci.json';

export default {
  name: 'Home',

  components: { Visualisation, Search, Words },

  data() {
    return {
      poslanci: poslanci.map((p) => {
        return {
          id: p.id,
          label: p.name,
        };
      }),
      selectedMPs: [],
    };
  },
  created() {
    this.$on('updateMPs', (mps) => {
      this.selectedMPs = mps;
    });
  },
  mounted() {
    console.log(poslanci);
  },
};
</script>
<style scoped lang="scss">
#master-search-container {
  position: fixed;
  z-index: 200;
  top: 10px;
  right: 10px;
  display: none;
}
</style>
