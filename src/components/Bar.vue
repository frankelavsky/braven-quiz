<template>
  <div class="bar">
    <span>({{Math.round((current / outOf)*1000)/10}}%{{message}})</span>
    <div class="gauge-background">
      <div v-bind:style="{width:Math.round((current / outOf)*1000)/10 + '%'}" class="gauge-fill"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Bar",
  props: {
    current: {
      default: 0,
      type: Number
    },
    outOf: {
      default: 0,
      type: Number
    },
    message: {
      default: "",
      type: String
    }
  },
  data: function() {
    return {
      count: this.points
    };
  }
};
</script>