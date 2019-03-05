<template>
  <div class="cars-results">
    <!-- Remove this?  -->
    <!-- <template v-for="(car, index) in cars">
      <p :key="index">
        Totalkostnad för
        <span class="highlight">{{ car.name }}</span> är
        <span class="highlight"> {{ formatNo(totalOwnershipCosts[index]) }} kr </span>
        varav {{ formatNo(totalFuelCosts[index]) }} kr i driftkostnad ({{
          (fuelCosts[index] * 10).toFixed(1).replace('.', ',')
        }}
        kr per mil)
        <span v-if="car.type === 'electric'"> och miljöbilspremie på {{ evBonus }} kr</span>
      </p>
    </template> -->

    <p>
      <span class="highlight">{{ cheapestInTotal.name }}</span> är billigast och utgör en
      <span class="highlight"> total besparing på {{ savingsFormatted }} kr </span>
      (eller {{ percentFormatted }}%) jämfört med {{ mostExpensiveIntotal.name }}.
      <span v-if="cheapestInTotal.type === 'electric'">
        Miljöbilspremien på {{ formatNo(evBonus) }} kr är inräknad.
      </span>
    </p>
    <p>
      {{ cheapestToRun.name }} {{ cheapestInTotal === cheapestToRun ? 'är också' : 'är dock' }}
      {{ fuelSavingsFormatted }} kr billigare i drift över {{ usage.ownership }} år och
      {{ distanceFormatted }} mil.
    </p>
  </div>
</template>
<script>
import { TweenLite } from 'gsap/TweenMax';
export default {
  props: ['usage', 'cars', 'evBonus'],
  data() {
    return {
      tweenedSavings: 0,
      tweenedPercent: 0,
      tweenedFuelSavings: 0,
      tweenedDistance: 0,
    };
  },
  mounted() {
    // Sets animation starting points
    this.tweenedSavings = this.totalSavings;
    this.tweenedPercent = this.totalSavingsPercent;
    this.tweenedFuelSavings = this.fuelSavings;
    this.tweenedDistance = this.totalDistance;
  },
  methods: {
    formatNo(num) {
      return Math.round(num).toLocaleString('sv-SE');
    },
  },
  computed: {
    fuelCosts: function() {
      const { gasPrice, kwhPrice } = this.usage;
      return this.cars.map(car => {
        const cost =
          car.type === 'electric'
            ? (car.consumption * kwhPrice) / 100
            : (car.consumption * gasPrice) / 100;
        return cost;
      });
    },
    totalFuelCosts: function() {
      const { distance, ownership } = this.usage;
      return this.cars.map((car, index) =>
        Math.round(this.fuelCosts[index] * distance * ownership),
      );
    },
    totalOwnershipCosts: function() {
      return this.cars.map((car, index) => {
        const cost = this.totalFuelCosts[index] + car.price;
        return car.type === 'electric' ? cost - this.evBonus : cost;
      });
    },
    totalDistance: function() {
      return this.usage.distance * this.usage.ownership;
    },
    fuelSavings: function() {
      const [carOne, carTwo] = this.totalFuelCosts;
      if (carOne < carTwo) return carTwo - carOne;
      else return carOne - carTwo;
    },
    totalSavings: function() {
      const [carOne, carTwo] = this.totalOwnershipCosts;
      if (carOne < carTwo) return carTwo - carOne;
      else return carOne - carTwo;
    },
    totalSavingsPercent: function() {
      const [carOne, carTwo] = this.totalOwnershipCosts;
      const diff = this.totalSavings;
      return Math.round(carOne > carTwo ? (diff / carOne) * 100 : (diff / carTwo) * 100);
    },
    cheapestToRun: function() {
      const [carOne, carTwo] = this.totalFuelCosts;
      return carOne < carTwo ? this.cars[0] : this.cars[1];
    },
    cheapestInTotal: function() {
      const [carOne, carTwo] = this.totalOwnershipCosts;
      return carOne < carTwo ? this.cars[0] : this.cars[1];
    },
    mostExpensiveToRun: function() {
      const [carOne, carTwo] = this.totalFuelCosts;
      return carOne > carTwo ? this.cars[0] : this.cars[1];
    },
    mostExpensiveIntotal: function() {
      const [carOne, carTwo] = this.totalOwnershipCosts;
      return carOne > carTwo ? this.cars[0] : this.cars[1];
    },
    energySaved: function() {
      return (this.mostExpensiveToRun.consumption / 100) * this.usage.distance;
    },
    // Formatted and tweened numbers for output below
    savingsFormatted: function() {
      return this.formatNo(this.tweenedSavings);
    },
    percentFormatted: function() {
      return this.formatNo(this.tweenedPercent);
    },
    fuelSavingsFormatted: function() {
      return this.formatNo(this.tweenedFuelSavings);
    },
    distanceFormatted: function() {
      return this.formatNo(this.tweenedDistance / 10);
    },
  },
  watch: {
    // Animates numbers on change
    totalSavings: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedSavings: newValue });
    },
    totalSavingsPercent: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedPercent: newValue });
    },
    fuelSavings: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedFuelSavings: newValue });
    },
    totalDistance: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedDistance: newValue });
    },
  },
};
</script>
<style lang="scss" scoped>
.cars-results {
  @include card-style;
  text-align: center;
  grid-area: results;
}
.highlight {
  font-size: 1.5rem;
  font-weight: bold;
  color: $accent-color;
  word-wrap: break-word;
}
</style>