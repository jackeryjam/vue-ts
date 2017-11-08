<template>
  <div class="hello">
    <b-row>
      <b-jumbotron class="text-center">
        <h1 class="display-3">{{title}}</h1>
        <p class="lead">{{tip}}</p>
        <hr class="my-4">
        <p>It uses utility classes for typography and spacing to space content out within the larger container.</p>
      </b-jumbotron>
    </b-row>
  </div>
</template>

<script lang="ts">
  import Vue from 'vue'
  import Component from 'vue-class-component'
  
  @Component
  export default class HelloWorld extends Vue {
    msg: string = 'this is a typescript project now'
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss"scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
$cell-side : 100px;
$queen-side : $cell-side * 0.9;
table td{
  width: $cell-side;
  height: $cell-side;
  text-align: center;
}
.cell{
  background: black;
}
.queen{
  width: $queen-side;
  height: $queen-side;
}
.jumbotron{
  width: 100%;
}
@for $i from 1 to 10 {
  .flex-#{$i}{
    flex: $i;
  }
}
</style>
