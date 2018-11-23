<template>
  <div id="calculator">
    <header>
      <h1>Calculator</h1>
      <span class="author">
        DESIGN
        <a href="https://www.freecodecamp.com/caiyiliang" target="_blank">CaiYiLiang</a>
      </span>
    </header>
    <header>
      <span class="author">
        FORKE
        <a href="#">TakuKitamura</a>
      </span>
    </header>
    <section id="calculator" class="calculator">
      <calculator-content/>
    </section>
  </div>
</template>

<script>
import CalculatorContent from '~/components/calculator.vue'

export default {
  components: { CalculatorContent }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Pacifico');
html {
  font-size: 62.5%; /* =10px */
}
body {
  background-color: #f4f1f1; /* HEX */
}
header {
  font-size: 1.8rem;
  font-family: 'Pacifico', cursive;
}
header .author {
  font-size: 2rem;
}
header .author a {
  color: #ad1457;
  text-decoration: none;
}
header,
.calculator {
  margin: 2.5rem auto;
  text-align: center;
}
.calculator {
  border: 0.6rem solid #37474f;
  width: 34rem;
  height: 40rem;
  background-color: #546e7a;
  font-size: 1.6rem;
  -webkit-border-radius: 1rem;
  -moz-border-radius: 1rem;
  -ms-border-radius: 1rem;
  -o-border-radius: 1rem;
  border-radius: 1rem;
  -webkit-box-shadow: 1rem 1rem 0.5rem #212121;
  box-shadow: 0.5rem 0.5rem 0.25rem #212121;
}
</style>