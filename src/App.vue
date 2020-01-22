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
.option-input,
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
  font-size: 3.3vw;
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
.results,
.main-explainer {
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
.results,
.main-explainer {
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
