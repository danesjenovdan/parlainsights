<template>
  <div id="text">
    <div id="intro">
      <h1>Vpogled v <strong>Parlameter</strong></h1>
      <p>Parlameter je v skoraj dveh letih delovanja zbral in anliziral podatke 7. sklica državnega zbora. Spodaj boš priča analizi, ki zajema vse poslanke in poslance preteklega sklica, v njej pa smo analizirali njihovo delo glede na nekatere skupne lastnosti.</p>
      <p>Če te kaka poslanka ali poslanec posebej zanima, lahko njeno/-ovo ime vpišeš v iskalnik in  jo/ga spremljaš skozi vse analize.</p>
      <search
        placeholder="placeholder"
        :items="poslanci"
      ></search>
    </div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="textbox">
      <p>Pozdravljen v pregledu preteklega sklica Državnega zbora RS!</p>
      <p>V tokratnem, sedmem sklicu sta poslanske sedeže zasedla 102 poslanca. V DZ RS sedi 90 ali manj poslancev. Nekateri med njimi postanejo ministri, drugi se vrnejo v poslanske klopi, spet tretji pa odidejo na prestajanje kazenskih sankcij.</p>
      <p>Od naših 102 poslancev jih je 70 funkcijo opravljalo vse dni sklica.</p>
    </div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
    <div class="spacer"></div>
  </div>
</template>
<script>
import Search from './Search';
export default {
    name: 'Words',
    props: {
      poslanci: Array,
    },
    components: {
      Search,
    },
    mounted() {
      this.$on('updateMPs', (mps) => {
        this.$parent.$emit('updateMPs', mps);
      });
    },
};
</script>
<style scoped lang="scss">
@import '../scss/colors';
@import url('https://fonts.googleapis.com/css?family=Roboto:300,400,900&subset=latin-ext');

#intro {
  h1, p {
    color: #ffffff;
  }
  background-color: #009cdd;
}

.spacer {
  width: 100%;
  height: 200px;
}

#text {
  z-index: 100;
  position: relative;
}
.textbox {
  background-color: $grey;
  padding: 10px;
  width: 80%;
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;

  p {
    margin: 0;
    padding: 10px;
    line-height: 1.3em;
    font-size: 16px;
    font-family: 'Roboto';
    font-weight: 300;
  }
}
</style>