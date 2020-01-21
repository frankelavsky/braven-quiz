<template>
  <div class="results">
    <h2 class="lead">Your Results: {{compact}}</h2>
    <p class="description">{{compact}} stands for {{type.Title}}.</p>
    <h3>Description</h3>
    <p>{{type.Description}}</p>
    <h3>Example Persona in Braven</h3>
    <p>{{type.Examples}}</p>
    <h2 class="breakdown-heading">Top Traits</h2>
    <p>(What you're great at)</p>
    <div v-for="top in topTraits" v-bind:key="top.trait">
      <h3>{{fullNames[top.trait]+": " + ranks[top.trait].value}}</h3>
      <Bar v-bind:current="results[top.trait]" v-bind:outOf="ranks[top.trait].pairTotal"></Bar>
      <p class="trait-explanation">{{traitResults[top.trait]}}</p>
    </div>
    <h2 class="breakdown-heading">Bottom Traits</h2>
    <p>(Tips for things to look out for)</p>
    <div v-for="bottom in topTraits" v-bind:key="bottom.trait">
      <h3>{{fullNames[pairing[bottom.trait]]+": " + ranks[pairing[bottom.trait]].value}}</h3>
      <Bar
        v-bind:current="results[pairing[bottom.trait]]"
        v-bind:outOf="ranks[pairing[bottom.trait]].pairTotal"
      ></Bar>
      <p class="trait-explanation">{{traitResults[pairing[bottom.trait]]}}</p>
    </div>
    <h2 class="breakdown-heading">Sibling Personalities</h2>
    <p>(Personalities you are similar to.)</p>
    <div v-for="bottom in topTraits" v-bind:key="bottom.trait">
      <h3>{{fullNames[pairing[bottom.trait]]+": " + ranks[pairing[bottom.trait]].value}}</h3>
      <Bar
        v-bind:current="results[pairing[bottom.trait]]"
        v-bind:outOf="ranks[pairing[bottom.trait]].pairTotal"
      ></Bar>
      <p class="trait-explanation">{{traitResults[pairing[bottom.trait]]}}</p>
    </div>
    <h2 class="breakdown-heading">Caveats</h2>
    <p>This personality system isn't perfect, of course. But the purpose of it is to provide a way for the Host (or DM) of your group to help you find a good role in your party, like a casting director would for a part in a movie or play. Both you and your teammates will be happiest if you are set up to play the best version of yourself possible. And a lot of that actually means working well with your teammates! Ask them their personality, see what they think of it, and let them know yours. It's a great way to break the ice too, if you are new to a group or playing a one-shot with unfamiliar folks.</p>
  </div>
</template>

<script>
import Bar from "./Bar.vue";
import Descriptions from "../assets/descriptions.json";
import ResultsJSON from "../assets/results.json";

export default {
  name: "Results",
  components: {
    Bar
  },
  props: {
    results: {
      default: () => {},
      type: Object
    }
  },
  data: function() {
    return {
      type: {
        default: () => {},
        type: Object
      },
      compact: {
        default: "",
        type: String
      },
      topTraits: {
        default: () => [],
        type: Array
      },
      ranks: {
        type: Object,
        default: () => {}
      },
      traitResults: {
        type: Object,
        default: () => {}
      },
      pairing: {
        type: Object,
        default: () => {}
      },
      fullNames: {
        type: Object,
        default: () => {}
      }
    };
  },
  created() {
    this.pairing = {
      C: "I",
      I: "C",
      R: "H",
      H: "R",
      D: "S",
      S: "D"
    };
    this.fullNames = {
      C: "Conventional",
      I: "Innovative",
      R: "Resolute",
      H: "Harmonious",
      D: "Deliberate",
      S: "Spontaneous"
    };
    this.rankTraits();
    const baseResults = {
      C: "",
      I: "",
      R: "",
      H: "",
      D: "",
      S: ""
    };
    Object.keys(baseResults).forEach(key => {
      baseResults[key] = ResultsJSON[0][key][this.ranks[key].value];
    });
    this.traitResults = baseResults;
  },
  methods: {
    rankTraits() {
      const traits = [];
      let name = "";
      traits.push({ trait: this.results.C >= this.results.I ? "C" : "I" });
      name += traits[0].trait;
      traits.push({ trait: this.results.R >= this.results.H ? "R" : "H" });
      name += traits[1].trait;
      traits.push({ trait: this.results.D >= this.results.S ? "D" : "S" });
      name += traits[2].trait;
      this.compact = name;
      this.topTraits = traits;
      this.type = Descriptions[this.compact];
      const rankedTraits = {
        C: {},
        I: {},
        R: {},
        H: {},
        D: {},
        S: {}
      };
      Object.keys(rankedTraits).forEach(key => {
        const pairTotal = this.results[key] + this.results[this.pairing[key]];
        rankedTraits.trait = key;
        rankedTraits[key].pairTotal = pairTotal;
        rankedTraits[key].similarity =
          this.results[key] / this.results[this.pairing[key]];
        if (key === "C" || key === "R" || key === "D") {
          rankedTraits[key].value =
            this.results[key] / pairTotal >= 0.75
              ? "High"
              : (rankedTraits[key].value =
                  this.results[key] / pairTotal >= 0.5 ? "Moderate" : "Low");
        } else {
          rankedTraits[key].value =
            this.results[key] / pairTotal > 0.75
              ? "High"
              : (rankedTraits[key].value =
                  this.results[key] / pairTotal > 0.5 ? "Moderate" : "Low");
        }
      });
      this.ranks = rankedTraits;
      // this.traits.forEach(trait=>{

      // })
      // this.type.similarities = {}

      // eslint-disable-next-line no-console
      console.log(
        this.compact,
        this.topTraits,
        this.results,
        this.ranks,
        Descriptions
      );
    }
  }
};
</script>