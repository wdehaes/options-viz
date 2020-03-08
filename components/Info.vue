<template>
  <div class="info">
    <h1 class="title">Options: Twitter example</h1>
    <p>Date: 01/20/2020 - Current Stock Price: $33.20</p>
    <p>
      profit: {{ profit }} = {{ stock }} (stock price) - {{ strike }} (strike
      price) - {{ price }} (option price)
    </p>
  </div>
</template>

<script>
export default {
  props: {
    price: {
      type: Number,
      default: () => 0
    },
    stock: {
      type: Number,
      default: () => 0
    },
    strike: {
      type: Number,
      default: () => 0
    }
  },
  computed: {
    profit() {
      return this.stock - this.strike - this.price
    }
  }
}
</script>

<style></style>
