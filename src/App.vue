<template>
  <div id="App">
    <h1>Braven's TTRPG Personality Quiz</h1>
    <p
      class="description"
    >Wouldn't it be awesome to know what kind of personality you have when you play a tabletop role-playing game? This quiz can help you become an amazing player, expand what types of characters you play well, and even improve your teamwork in a party.</p>
    <div class="main-content-wrapper">
      <div v-if="explanation" class="main-explainer">
        <h2>Braven's Personality System</h2>
        <p>
          In Braven, part of making a character involves taking a personality quiz. This helps players learn more about themselves at the table but also helps them learn what their teammates are like too. Choosing your personality is hard.
          <a
            href="https://bravengames.wordpress.com/2019/12/13/picking-your-personality/"
            target="_blank"
          >We write about this, if you want to know more in-depth</a>.
        </p>
        <p>
          <br />In short: there are three spectrums we quiz across (outlined below). And before I explain these, just know: all personalities are valid! And in fact, a good party has a mix of personalities that play well off of each others strengths and weaknesses. "Aversion" to something is not bad. I really hope to state that clearly before you dig in!
        </p>
        <h3>Innovative (I) versus Conventional (C)</h3>
        <p>This spectrum is your aversion to risk.</p>
        <h3>Resolute (R) versus Harmonious (H)</h3>
        <p>This spectrum is your aversion to interpersonal conflict.</p>
        <h3>Deliberate (D) versus Spontaneous (S)</h3>
        <p>This spectrum is your aversion to order.</p>
        <p>
          <br />In the end, you'll get an explanation of your results, some tips, and an acronym that represents one of the 8 possible outcomes from this quiz. That acronym is your TTRPG personality! Feel free to say things like "Yeah I'm kind of an IHD myself" (or whatever floats your boat).
        </p>
        <p>
          <br />There are 8 possible outcomes, but if you tie or get close, the quiz results will suggest other acronyms to look at. As mentioned previously, if you want to know more you can check out our
          <a
            href="https://bravengames.wordpress.com/2019/12/13/picking-your-personality/"
            target="_blank"
          >blog post</a>. Otherwise, you can just
        </p>
        <button v-on:click="next">Begin the quiz!</button>
      </div>
      <div v-else>
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
    </div>
    <div class="footer">
      <div v-if="resultsReady" class="button-wrapper">
        <span>
          Not the results you are looking for?
          <a
            v-bind:href="pageSource"
            class="light"
            target="_blank"
          >Take the test</a>
        </span>
      </div>
      <div v-else class="button-wrapper">
        <button
          v-on:click="back"
          v-bind:disabled="!(!explanation || currentPage)"
        >{{!explanation && !currentPage ? "Back to Explanation" : "Back"}}</button>
        <button
          v-on:click="next"
          v-bind:disabled="!nextReady"
        >{{currentPage === shuffledQuestions.length-1 ? "SUBMIT" : explanation ? "Begin the quiz!" : "Next"}}</button>
      </div>
      <div class="learn-more">
        We don't steal, store, or sell any data when you visit us (and never will).
        <a
          class="light"
          href="https://github.com/frankelavsky/braven-quiz"
          target="_blank"
        >The code for this quiz is open source.</a>
        <br />Like this?
        <a
          class="light"
          href="https://paypal.me/FrankElavsky"
          target="_blank"
        >Buy me a coffee</a> or
        <a
          class="light"
          href="https://bravengames.wordpress.com/"
          target="_blank"
        >follow our blog</a> to learn more about Braven, our tabletop roleplaying game.
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
      nextReady: true,
      resultsReady: false,
      explanation: true,
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
    back() {
      if (!this.explanation && !this.currentPage) {
        this.explanation = true;
        this.nextReady = true;
      } else {
        this.currentPage--;
      }
    },
    next() {
      if (this.explanation) {
        this.explanation = false;
        this.nextReady = false;
      } else if (
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
          this.prepURLParams();
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
      };
      Object.keys(this.pageResults).forEach(key => {
        Object.keys(this.pageResults[key]).forEach(trait => {
          newResults[trait] += this.pageResults[key][trait];
        });
      });
      this.resultsReady = true;
      this.aggregatedResults = newResults;
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
      if (entryPoints !== totalPoints) {
        window.location.search = "";
      } else {
        this.explanation = false;
        this.resultsReady = true;
        this.aggregatedResults = urlResults;
      }
    }
  }
};
</script>

<style>
@media all and (min-width: 340px) {
  :root {
    --small-spacing: 1vw;
    --moderate-spacing: 3vw;
    --large-spacing: 5vw;
    --footer-spacing: 26vw;
    --font-small: 3vw;
    --font-regular: 3.3vw;
    --font-emphasized: 3.5vw;
    --font-button: 3.8vw;
    --font-h2: 3.8vw;
    --font-h1: 4.2vw;
  }
}
@media all and (min-width: 480px) {
  :root {
    --small-spacing: 1vw;
    --moderate-spacing: 3vw;
    --large-spacing: 5vw;
    --footer-spacing: 26vw;
    --font-small: 3vw;
    --font-regular: 3.3vw;
    --font-emphasized: 3.5vw;
    --font-button: 3.8vw;
    --font-h2: 3.8vw;
    --font-h1: 4.2vw;
  }
}
@media all and (min-width: 768px) {
  :root {
    --small-spacing: 0.75vw;
    --moderate-spacing: 2.25vw;
    --large-spacing: 7vw;
    --footer-spacing: 22vw;
    --font-small: 2.5vw;
    --font-regular: 2.8vw;
    --font-emphasized: 3vw;
    --font-button: 3.3vw;
    --font-h2: 3.3vw;
    --font-h1: 3.7vw;
  }
}
@media all and (min-width: 1200px) {
  :root {
    --small-spacing: 0.5vw;
    --moderate-spacing: 1.5vw;
    --large-spacing: 12vw;
    --footer-spacing: 18vw;
    --font-small: 1.8vw;
    --font-regular: 2vw;
    --font-emphasized: 2.3vw;
    --font-button: 2.3vw;
    --font-h2: 2.8vw;
    --font-h1: 3.2vw;
  }
}
@media all and (min-width: 1440px) {
  :root {
    --small-spacing: 0.3vw;
    --moderate-spacing: 1vw;
    --large-spacing: 16vw;
    --footer-spacing: 10vw;
    --font-small: 1.4vw;
    --font-regular: 1.6vw;
    --font-emphasized: 1.8vw;
    --font-button: 1.8vw;
    --font-h2: 2.4vw;
    --font-h1: 3vw;
  }
}
@media all and (min-width: 1920px) {
  :root {
    --small-spacing: 0.3vw;
    --moderate-spacing: 1vw;
    --large-spacing: 20vw;
    --footer-spacing: 8vw;
    --font-small: 1.2vw;
    --font-regular: 1.4vw;
    --font-emphasized: 1.6vw;
    --font-button: 1.6vw;
    --font-h2: 2.4vw;
    --font-h1: 3vw;
  }
}
#App {
  font-family: "Vesper Libre", serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #313131;
  margin-top: calc(var(--small-spacing) * 2);
  font-size: var(--font-regular);
}
h1,
h2,
button,
.option-input,
.points {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  font-weight: 900;
}
h3 {
  font-family: "Montserrat", "Avenir", Helvetica, Arial, sans-serif;
  font-weight: 300;
  margin-block-start: var(--small-spacing);
}
h1 {
  font-size: var(--font-h1);
}
h2 {
  font-size: var(--font-h2);
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
  padding: 0vw var(--large-spacing) 0vw var(--large-spacing);
}
.panel,
.results,
.main-explainer {
  margin: var(--moderate-spacing) var(--large-spacing);
  padding: calc(var(--small-spacing) * 2);
}
.button-wrapper {
  margin: 0 var(--large-spacing);
  padding: var(--small-spacing) calc(var(--small-spacing) * 2);
}
.main-content-wrapper {
  margin-bottom: var(--footer-spacing);
}
.options-table {
  width: 100%;
}
.panel,
.results,
.main-explainer {
  background: #eeeeee;
  border-radius: calc(var(--small-spacing) * 2);
}
.remaining-points {
  padding-bottom: var(--small-spacing);
  color: #747474;
}
.points {
  color: #313131;
  font-size: var(--font-emphasized);
}
.gauge-background {
  height: calc(var(--small-spacing) * 2);
  width: 100%;
  position: relative;
  border-radius: var(--small-spacing);
  background: white;
}
.gauge-fill {
  background: #959595;
  left: 0;
  height: calc(var(--small-spacing) * 2);
  position: relative;
  border-radius: var(--small-spacing) 0vw 0vw var(--small-spacing);
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
  font-size: var(--font-button);
  border-radius: var(--small-spacing);
  width: calc(46vw - var(--large-spacing));
  padding: 0px;
  padding-top: calc(var(--small-spacing) / 2);
}
button:disabled,
button[disabled] {
  color: #6b6b6b;
  background: #eeeeee;
}
.progress {
  padding-top: var(--small-spacing);
}
.bar {
  color: #747474;
}
.input-cell {
  width: 20%;
  padding: calc(var(--small-spacing) * 1.5) 0;
}
.option-input {
  width: 50%;
  height: var(--moderate-spacing);
  color: #313131;
  font-size: var(--font-emphasized);
  padding: calc(var(--small-spacing) * 2);
  border-radius: calc(var(--small-spacing) / 2);
}
.learn-more {
  padding: 0vw var(--large-spacing);
}
.option-text {
  text-align: right;
  line-height: calc(var(--font-regular) * 1.5);
}
.description,
.learn-more,
p {
  line-height: calc(var(--font-small) * 1.5);
  font-size: var(--font-small);
}
.trait-explanation {
  padding: var(--small-spacing) 0vw var(--small-spacing) 0vw;
}
.breakdown-heading {
  padding-top: calc(var(--small-spacing) * 2);
}
a {
  color: #006dac;
}
a:visited {
  color: #94007c;
}
a.light {
  color: #00c3ff;
}
a.light:visited {
  color: #ff97ee;
}
</style>
