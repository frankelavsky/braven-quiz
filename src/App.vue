<template>
  <div id="App">
    <h1>Braven's TTRPG Personality Quiz</h1>
    <p
      class="description"
    >Wouldn't it be awesome to know what kind of personality you have when you play a tabletop role-playing game? This quiz can help you learn more about yourself and your teammates, so that you can become an amazing party together.</p>
    <div class="main-content-wrapper">
      <div v-if="resultsReady">
        <Results v-bind:results="aggregatedResults"></Results>
      </div>
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
    </div>
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
      <div class="learn-more">
        We don't steal, store, or sell any data when you visit us (and never will).
        <a
          href="https://github.com/frankelavsky/braven-quiz"
          target="_blank"
        >The code for this quiz is open source.</a>
        <br />Like this?
        <a href="https://paypal.me/FrankElavsky" target="_blank">Buy me a coffee</a> or
        <a href="https://bravengames.wordpress.com/" target="_blank">follow our blog</a> to learn more about Braven, our Tabletop Roleplaying Game.
      </div>
    </div>
  </div>
</template>

<script>
import Panel from "./components/Panel.vue";
import Results from "./components/Results.vue";
import Questions from "./assets/questions.json";

export default {
  name: "App",
  components: {
    Panel,
    Results
  },
  data() {
    return {
      shuffledQuestions: [],
      currentPage: 0,
      pageResults: {},
      nextReady: false,
      resultsReady: false,
      aggregatedResults: {},
      pageSource: "/" // window.location.protocol + window.location.host + window.location.pathname
    };
  },
  async created() {
    if (window.location.search) {
      this.parseURLParams();
    } else {
      Questions.forEach(question => {
        this.shuffle(question.options);
      });
      this.shuffledQuestions = this.shuffle(Questions);
    }
  },
  methods: {
    shuffle(a) {
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [a[i], a[j]] = [a[j], a[i]];
      }
      return a;
    },
    readyNext(ready) {
      this.pageResults[ready.page] = { ...ready.results };
      this.nextReady = true;
    },
    disableNext() {
      // disable next button
      this.nextReady = false;
    },
    next() {
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
      this.resultsReady = true;
      this.aggregatedResults = newResults;
      this.prepURLParams();
      // eslint-disable-next-line no-console
      console.log("PREPPED results", this.aggregatedResults);
    },
    warnUserNoGoingBack() {
      return confirm(
        "You are about to view your results.\n\nYou will not be able to edit your responses once you proceed."
      );
    },
    prepURLParams() {
      let params = "?";
      let count = 0;
      Object.keys(this.aggregatedResults).forEach(key => {
        params += count ? "&" : "";
        params += key + "=" + this.aggregatedResults[key];
        count++;
      });
      window.history.pushState({}, document.title, params);
    },
    parseURLParams() {
      const url = new URL(window.location.href);
      const entries = [...url.searchParams.entries()];
      let totalPoints = 0;
      Questions.forEach(question => {
        totalPoints += question.points;
      });
      let entryPoints = 0;
      const urlResults = {};
      entries.forEach(entry => {
        entryPoints += +entry[1];
        urlResults[entry[0]] = +entry[1];
      });
      // eslint-disable-next-line no-console
      console.log("entrypoints now ", entryPoints, totalPoints, urlResults);
      if (entryPoints !== totalPoints) {
        window.location.search = "";
      } else {
        this.resultsReady = true;
        this.aggregatedResults = urlResults;
      }
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
button,
.option-iniput,
.points {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  font-weight: 900;
}
h3 {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  font-weight: 300;
  margin-block-start: 1vw;
}
h1 {
  font-size: 3vw;
}
h2 {
  font-size: 2.6vw;
}
h2,
p {
  margin-block-start: 0;
}
h1,
h2,
h3,
p {
  margin-block-end: 0;
}
.description {
  padding: 0vw 10vw 0vw 10vw;
}
.panel,
.results {
  margin: 3vw 10vw;
  padding: 2vw;
}
.button-wrapper {
  margin: 0 10vw;
  padding: 1vw 2vw;
}
.main-content-wrapper {
  margin-bottom: 10vw;
}
.panel,
.results {
  background: #eeeeee;
  border-radius: 2vw;
}
.remaining-points {
  padding-bottom: 1vw;
  color: #747474;
}
.points {
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
  background: #313131e0;
  color: white;
}
button {
  cursor: pointer;
  color: #ffffff;
  background: #0f63ff;
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
}
.bar {
  color: #747474;
}
.option-input {
  width: 3vw;
  height: 2vw;
  color: #313131;
  font-size: 1.8vw;
  padding: 0.2vw 0.5vw;
  margin-left: 1.5vw;
  border-radius: 0.5vw;
}
.learn-more {
  padding: 0vw 10vw;
}
.description,
.learn-more,
p {
  line-height: 1.8vw;
  font-size: 1.2vw;
}
.trait-explanation {
  padding: 1vw 0vw 1vw 0vw;
}
.breakdown-heading {
  padding-top: 2vw;
}
a {
  color: #00c3ff;
}
a:visited {
  color: #ff97ee;
}
</style>
