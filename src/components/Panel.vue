<template>
  <div class="panel">
    <h2 class="question">{{question}}</h2>
    <div class="remaining-points">
      (Spend these points among the options:
      <span class="points">{{" " + count}}</span>).
    </div>
    <form v-on:submit.prevent="submit()">
      <input type="submit" style="display: none" />
      <div v-for="option in options" v-bind:key="option.trait" class="option">
        <span class="option-text">{{option.text}}</span>
        <input
          v-on:input="calculateCount($event.target,$event.target.value,option.trait)"
          v-bind:max="points"
          v-bind:value="results[option.trait] ? results[option.trait] : previousResults[option.trait] ? previousResults[option.trait] : 0"
          class="option-input"
          type="number"
          min="0"
        />
      </div>
    </form>
    <div class="progress">
      <span>({{Math.round((page / totalPages)*1000)/10}}% finished with the quiz.)</span>
      <div class="gauge-background">
        <div
          v-bind:style="{width:Math.round((page / totalPages)*1000)/10 + '%'}"
          class="gauge-fill"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Panel",
  props: {
    question: {
      default: "",
      type: String
    },
    options: {
      default: () => [],
      type: Array
    },
    page: {
      default: 0,
      type: Number
    },
    totalPages: {
      default: 0,
      type: Number
    },
    points: {
      default: 0,
      type: Number
    },
    type: {
      default: "",
      type: String
    },
    previousResults: {
      default: () => {},
      type: Object
    }
  },
  data: function() {
    return {
      count: this.points,
      results: {}
    };
  },
  watch: {
    points(n) {
      this.count = n;
    },
    options(n) {
      this.setFreshSlate(n);
    }
  },
  created() {
    // eslint-disable-next-line no-console
    console.log("we have loaded", this.results, this.options);
    this.setFreshSlate(this.options);
  },
  methods: {
    submit() {
      this.$emit("submitToNext");
    },
    setFreshSlate(optionsArray) {
      if (!Object.keys(this.previousResults).length) {
        this.count = this.points;
        const freshSlate = {};
        optionsArray.forEach(option => {
          freshSlate[option.trait] = 0;
        });
        this.results = freshSlate;
        this.$emit("notReady");
      } else {
        this.count = 0;
        this.results = { ...this.previousResults };
        this.$emit("ready", {
          results: { ...this.previousResults },
          page: this.page
        });
      }
    },
    calculateCount(element, inputValue, trait) {
      let value = inputValue;
      if (value < 0) {
        value = 0;
        element.value = 0;
      } else if (value - this.results[trait] > this.count) {
        value = this.results[trait] + this.count;
        element.value = this.results[trait] + this.count;
      }
      this.results[trait] = +value;
      let remaining = this.points;
      Object.keys(this.results).forEach(key => {
        remaining -= this.results[key];
      });
      this.count = remaining;
      if (this.count === 0) {
        this.$emit("ready", { results: { ...this.results }, page: this.page });
      }
    }
  }
};
</script>