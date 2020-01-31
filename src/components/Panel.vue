<template>
  <div class="panel">
    <h2 class="question">{{question}}</h2>
    <div class="remaining-points">
      (Spend these points among the options:
      <span class="points">{{" " + count}}</span>).
    </div>
    <form v-on:submit.prevent="submit()">
      <input type="submit" style="display: none" />
      <table class="options-table">
        <tr v-for="(option, index) in options" v-bind:key="option.trait" class="option">
          <td class="option-text">{{option.text}}</td>
          <td class="input-cell">
            <input
              v-on:input="calculateCount($event.target,$event.target.value,option.trait)"
              v-bind:max="points"
              v-bind:value="results[option.trait] || results[option.trait] === 0 || results[option.trait] === '' ? results[option.trait] : previousResults[option.trait] ? previousResults[option.trait] : 0"
              v-bind:ref="'input'+index"
              class="option-input"
              type="number"
              min="0"
            />
          </td>
        </tr>
      </table>
    </form>
    <Bar
      v-bind:current="page"
      v-bind:outOf="totalPages"
      message=" finished with the quiz."
      class="progress"
    ></Bar>
  </div>
</template>

<script>
import Bar from "./Bar.vue";

export default {
  name: "Panel",
  components: {
    Bar
  },
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
    this.setFreshSlate(this.options);
  },
  methods: {
    submit() {
      this.$emit("submitToNext");
    },
    focusFirstInput() {
      this.$refs.input0[0].focus();
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
      this.$nextTick(() => {
        this.focusFirstInput();
      });
    },
    calculateCount(element, inputValue, trait) {
      let value = inputValue;
      if (+value < 0) {
        value = 0;
      } else if (+value - +this.results[trait] > this.count) {
        value = this.results[trait] + this.count;
      }
      this.results[trait] = value === "" ? value : Math.round(+value);
      element.value = this.results[trait];

      let remaining = this.points;
      Object.keys(this.results).forEach(key => {
        remaining -= this.results[key];
      });
      this.count = remaining;

      if (this.count === 0) {
        const numberedResults = {};
        Object.keys(this.results).forEach(key => {
          numberedResults[key] = +this.results[key];
        });
        this.$emit("ready", { results: numberedResults, page: this.page });
      } else {
        this.$emit("notReady");
      }
    }
  }
};
</script>