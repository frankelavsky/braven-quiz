<template>
  <div class="panel">
    <h2 class="question">{{question}}</h2>
    <div class="remaining-points">
      (Spend these points among the options:
      <span class="points">{{" " + count}}</span>).
    </div>
    <div v-for="option in options" v-bind:key="option.trait" class="option">
      <span class="option-text">{{option.text}}</span>
      <input
        id="myNumber"
        v-on:input="calculateCount($event.target,$event.target.value,option.trait)"
        v-bind:max="points"
        v-bind:value="results[option.trait] ? results[option.trait] : 0"
        class="option-input"
        type="number"
        min="0"
      />
    </div>
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
      const freshSlate = {};
      n.forEach(option => {
        freshSlate[option.trait] = 0;
      });
      this.count = this.points;
      this.results = freshSlate;
      this.$emit("notReady");
    }
  },
  methods: {
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
      // eslint-disable-next-line no-console
      // console.log(this.results);
      let remaining = this.points;
      Object.keys(this.results).forEach(key => {
        remaining -= this.results[key];
      });
      this.count = remaining;
      if (this.count === 0) {
        // eslint-disable-next-line no-console
        console.log("zero!", { ...this.results }, this.page);
        this.$emit("ready", { results: { ...this.results }, page: this.page });
      }
    }
  }
};
</script>