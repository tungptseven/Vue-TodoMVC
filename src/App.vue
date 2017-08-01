<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view></router-view>
    <ul id="link_footer">
      <li v-for="item in links">
        <router-link v-bind:to=item.link>{{ item.desc }}</router-link>
      </li>
    </ul>
  </div>
</template>

<script>

  export default {
    name: 'app',
    data() {
      return {
        links: [{
            link: '/',
            desc: 'Hello World',
            done: true
          },
          {
            link: '/todo-mvc/all',
            desc: 'Todo MVC',
            done: true
          },
        ]
      }
    },
  }

</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  ul#link_footer {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

</style>
