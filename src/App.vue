<template>
  <div id="App">
    <Panel
      v-bind:page="currentPage"
      v-bind:totalPages="shuffledQuestions.length"
      v-bind:options="shuffledQuestions[currentPage].options"
      v-bind:question="shuffledQuestions[currentPage].question"
      v-bind:points="shuffledQuestions[currentPage].points"
    />

    <div class="footer">
      <div class="button-wrapper">
        <button v-on:click="currentPage--">Back</button>
        <button v-on:click="currentPage++">Next</button>
      </div>
    </div>
  </div>
</template>

<script>
import Panel from "./components/Panel.vue";
import Questions from "./assets/questions.json";

export default {
  name: "App",
  components: {
    Panel
  },
  data() {
    return {
      shuffledQuestions: [],
      currentPage: 0
    };
  },
  async created() {
    function shuffle(a) {
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [a[i], a[j]] = [a[j], a[i]];
      }
      return a;
    }
    Questions.forEach(question => {
      shuffle(question.options);
    });
    this.shuffledQuestions = shuffle(Questions);
  }
};
</script>

<style>
#App {
  font-family: "Vesper Libre", serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #313131;
  margin-top: 5vw;
  font-size: 1.8vw;
}
h1,
h2,
button {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
}
h1 {
  font-size: 3.5vw;
}
h2 {
  font-size: 3.2vw;
}
.button-wrapper,
.panel {
  margin: 3vw 10vw;
  padding: 2vw;
}
.panel {
  background: #eeeeee;
  border-radius: 2vw;
}
.remaining-points {
  padding-bottom: 1vw;
  color: #747474;
}
.points {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  color: #313131;
  font-size: 2vw;
}
.gauge-background {
  height: 20px;
  width: 100%;
  position: relative;
  border-radius: 10px;
  background: white;
}
.gauge-fill {
  background: #959595;
  left: 0;
  height: 20px;
  position: relative;
  border-radius: 10px 0px 0px 10px;
}
.footer {
  position: fixed;
  bottom: 0;
  text-align: center;
  margin: 0 auto;
}
button {
  color: #313131;
  background: #dde0eb;
  font-family: "Vesper Libre", serif;
  font-size: 2.3vw;
  line-height: 3vw;
  border-radius: 1vw;
  width: 37.5vw;
  padding: 0px;
  padding-top: 0.65vw;
}
.progress {
  padding-top: 1vw;
  color: #747474;
}
.option {
  padding: 0.05vw;
}
.option-input {
  width: 2vw;
  height: 2vw;
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  color: #313131;
  font-size: 2vw;
  padding: 0.5vw;
  margin-left: 1.5vw;
  border-radius: 0.5vw;
}
</style>
