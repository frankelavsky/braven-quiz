<template>
  <div id="App">
    <div v-if="resultsReady"></div>
    <Panel
      v-else
      v-bind:page="currentPage"
      v-bind:totalPages="shuffledQuestions.length"
      v-bind:options="shuffledQuestions[currentPage].options"
      v-bind:question="shuffledQuestions[currentPage].question"
      v-bind:points="shuffledQuestions[currentPage].points"
      v-bind:previousResults="pageResults[currentPage] || {}"
      @ready="readyNext"
      @notReady="disableNext"
      @submitToNext="next"
    />

    <div class="footer">
      <div v-if="resultsReady" class="button-wrapper">
        <span>
          Not the results you are looking for?
          <a
            v-bind:href="pageSource"
            target="_blank"
          >Take the test</a>
        </span>
      </div>
      <div v-else class="button-wrapper">
        <button v-on:click="currentPage--" v-bind:disabled="!currentPage">Back</button>
        <button
          v-on:click="next"
          v-bind:disabled="!nextReady"
        >{{currentPage === shuffledQuestions.length-1 ? "SUBMIT" : "Next"}}</button>
      </div>
      <a
        class="learn-more"
        href="https://bravengames.wordpress.com/"
        target="_blank"
      >Learn more about Braven, the Tabletop Roleplaying Game</a>
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
      currentPage: 0,
      pageResults: {},
      nextReady: false,
      resultsReady: false,
      aggregatedResults: {},
      pageSource:
        window.location.protocol +
        window.location.host +
        window.location.pathname
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
  },
  methods: {
    readyNext(ready) {
      this.pageResults[ready.page] = { ...ready.results };
      this.nextReady = true;
    },
    disableNext() {
      // disable next button
      this.nextReady = false;
    },
    next() {
      // eslint-disable-next-line no-console
      console.log(
        "current",
        this.currentPage,
        "end",
        this.shuffledQuestions.length - 1
      );
      if (
        this.currentPage < this.shuffledQuestions.length - 1 &&
        this.nextReady
      ) {
        this.currentPage++;
      } else if (
        this.nextReady &&
        this.currentPage === this.shuffledQuestions.length - 1
      ) {
        const userConfirmation = this.warnUserNoGoingBack();
        if (userConfirmation) {
          this.createAggregatedResults();
          this.resultsReady = true;
        }
      }
    },
    createAggregatedResults() {
      const newResults = {
        I: 0,
        C: 0,
        H: 0,
        R: 0,
        S: 0,
        D: 0
        // I: 11,
        // C: 2,
        // H: 9,
        // R: 6,
        // S: 6,
        // D: 6
      };
      Object.keys(this.pageResults).forEach(key => {
        Object.keys(this.pageResults[key]).forEach(trait => {
          newResults[trait] += this.pageResults[key][trait];
        });
      });
      this.aggregatedResults = newResults;
      // eslint-disable-next-line no-console
      console.log("PREPPED results", this.aggregatedResults);
    },
    warnUserNoGoingBack() {
      return confirm(
        "You are about to view your results.\n\nYou will not be able to edit your responses once you proceed."
      );
    }
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
  margin-top: 2vw;
  font-size: 1.6vw;
}
h1,
h2,
button {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
}
h1 {
  font-size: 3vw;
}
h2 {
  margin-block-start: 0;
  font-size: 2.6vw;
}
.button-wrapper,
.panel {
  margin: 3vw 10vw;
  padding: 2vw;
}
.button-wrapper {
  margin-bottom: 0;
  padding-bottom: 0;
}
.panel {
  background: #eeeeee;
  border-radius: 2vw;
  margin-bottom: 5vw;
}
.remaining-points {
  padding-bottom: 1vw;
  color: #747474;
}
.points {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  color: #313131;
  font-size: 1.8vw;
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
  width: 100%;
  position: fixed;
  bottom: 0;
  text-align: center;
  margin: 0 auto;
}
button {
  cursor: pointer;
  color: #313131;
  background: #dde0eb;
  font-family: "Vesper Libre", serif;
  font-size: 2vw;
  line-height: 2.6vw;
  border-radius: 1vw;
  width: 37vw;
  padding: 0px;
  padding-top: 0.65vw;
}
button:disabled,
button[disabled] {
  color: #6b6b6b;
  background: #eeeeee;
}
.progress {
  padding-top: 1vw;
  color: #747474;
}
.option-input {
  width: 3vw;
  height: 2vw;
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  color: #313131;
  font-size: 1.8vw;
  padding: 0.5vw;
  margin-left: 1.5vw;
  border-radius: 0.5vw;
}
.learn-more {
  font-size: 1.2vw;
}
</style>
