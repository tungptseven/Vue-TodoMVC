<template>
  <div>
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
      </header>
      <section class="main" v-show="todos.length" v-cloak>
        <input class="toggle-all" type="checkbox" v-model="allDone">
        <ul class="todo-list">
          <li v-for="todo in filteredTodos" class="todo" :key="todo.id" :class="{ completed: todo.completed, editing: todo == editedTodo }">
            <div class="view">
              <input class="toggle" type="checkbox" @click="completeTodo(todo)">
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo.id)"></button>
            </div>
            <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)">
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="todos.length" v-cloak>
        <span class="todo-count"><strong>{{ remaining }}</strong> {{ remaining | pluralize }} left</span>
        <ul class="filters">
          <li>
            <router-link :to="{name: 'Todo', params: {status: 'all'}}" :class="{ selected: visibility == 'all' }">All</router-link>
          </li>
          <li>
            <router-link :to="{name: 'Todo', params: {status: 'active'}}" :class="{ selected: visibility == 'active' }">Active</router-link>
          </li>
          <li>
            <router-link :to="{name: 'Todo', params: {status: 'completed'}}" :class="{ selected: visibility == 'completed' }">Completed</router-link>
          </li>
        </ul>
        <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">Clear completed</button>
      </footer>
    </section>
    <todo-footer></todo-footer>
  </div>
</template>

<script>
  // ---css---
  import '../assets/app.js';
  import Footer from './Footer.vue';
  import axios from 'axios';

  export default {

    data() {
      return {
        todos: [],
        newTodo: '',
        editedTodo: null,
        visibility: 'all',
        filters: null,
        user: '',
        showTodo: false
      }
    },
    created() {
      // visibility filters
      this.filters = {
        all: function (todos) {
          return todos
        },
        active: function (todos) {
          return todos.filter(function (todo) {
            return !todo.completed
          })
        },
        completed: function (todos) {
          return todos.filter(function (todo) {
            return todo.completed
          })
        }
      }
      // retrieve todo data from server
      axios.get(`http://192.168.1.107:3000/api/todo-mvc/all`)
        .then(res => {
          this.todos = res.data
          this.visibility = this.$route.params.status
        })
        .catch(error => {
          console.error(error)
        })
    },
    // watch todos change for localStorage persistence
    watch: {
      todos: {
        handler: function (todos) {}
      },
      '$route' (to, from) {
        console.log('Change route ', this.$route.params.status);
        this.visibility = this.$route.params.status;
      }
    },

    // computed properties http://vuejs.org/guide/computed.html
    computed: {
      filteredTodos: function () {
        return this.filters[this.visibility](this.todos)
      },
      remaining: function () {
        return this.filters
          .active(this.todos)
          .length
      },
      allDone: {
        get: function () {
          return this.remaining === 0
        },
        set: function (value) {
          this
            .todos
            .forEach(function (todo) {
              todo.completed = value
            })
        }
      }
    },

    filters: {
      pluralize: function (n) {
        return n === 1 ?
          'item' :
          'items'
      }
    },

    // methods that implement data logic. note there's no DOM manipulation here at
    // all.
    methods: {

      addTodo: function () {
        var value = this.newTodo && this
          .newTodo
          .trim()
        if (!value) {
          return
        }
        axios.post('http://192.168.1.107:3000/api/todo-mvc/addTodo', {
            title: value,
            completed: false
          })
          .then(res => {
            // this.todos.push(res.data[0])
            this.todos = res.data
            this.newTodo = ''
          })
          .catch(err => {
            console.log(err)
          })
      },

      removeTodo: function (id) {
        axios.post('http://192.168.1.107:3000/api/todo-mvc/removeTodo', {
            id
          })
          .then(res => {
            /*
            let todo = this.todos.find(todo => {
              return todo.id === res.data[0].id
            })
            this.todos.splice(this.todos.indexOf(todo), 1)
            */
            this.todos = res.data
          })
          .catch(err => {
            console.log(err)
          })
      },

      completeTodo: function (todo) {
        //todo.completed = !todo.completed
        axios.post('http://192.168.1.107:3000/api/todo-mvc/completeTodo', {
            todo
          })
          .then(res => {
            /*
            let todo = this.todos.find(todo => {
              return todo.id === res.data[0].id
            })
            this.todos.splice(this.todos.indexOf(todo), 1, res.data[0])
            */
            
            this.todos = res.data
            this.editedTodo = null
          })
          .catch(err => {
            console.log(err)
          })
      },

      editTodo: function (todo) {
        this.beforeEditCache = todo.title
        this.editedTodo = todo
      },

      doneEdit: function (todo) {
        if (!this.editedTodo || !this.editedTodo.title) {
          // todo.title = this.beforeEditCache
          return
        }
        axios.post('http://192.168.1.107:3000/api/todo-mvc/editTodo', {
            todo
          })
          .then(res => {
            /*
            let todo = this.todos.find(todo => {
              return todo.id === res.data[0].id
            })
            this.todos.splice(this.todos.indexOf(todo), 1, res.data[0])
            */
            this.todos = res.data
            this.editedTodo = null
          })
          .catch(err => {
            console.log(err)
          })
      },

      cancelEdit: function (todo) {
        this.editedTodo = null
        todo.title = this.beforeEditCache
      },

      removeCompleted: function () {
        axios.post('http://192.168.1.107:3000/api/todo-mvc/removeCompleted', {
            completedTodos: this.filters.completed(this.todos)
          })
          .then(res => {
            /*
            this.todos = this.todos.filter(todo => {
              return res.data.indexOf(todo.id) === -1;
            });
            */
            this.todos = res.data
          })
          .catch(err => {
            console.log(err)
          })
      }
    },

    // a custom directive to wait for the DOM to be updated before focusing on the
    // input field. http://vuejs.org/guide/custom-directive.html
    directives: {
      'todo-focus': function (el, binding) {
        if (binding.value) {
          el.focus()
        }
      }
    },
    components: {
      todoFooter: Footer
    }
  }

</script>

<style scoped>
  .todo-list li {
    display: block;
  }

</style>
