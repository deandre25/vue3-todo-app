<script>
import { getTodos, createTodo, updateTodo, deleteTodo } from './api/todos'
import Message from './components/Message.vue';
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';

export default {
  components: {
    StatusFilter,
    TodoItem,
    Message
},
  data() {
    return {
      todos: [],
      title: '',
      status: 'all',
      errorMessage: '',
    }
  },
  computed: {
    activeTodos() {
      return this.todos.filter(todo => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter(todo => todo.completed);
    },
    visibleTodos() {
      switch (this.status) {
        case 'active':
          return this.activeTodos;
          
        case 'completed':
          return  this.completedTodos;
        
        default:
          return this.todos;
      }
    }
  },
  mounted() {
    getTodos()
      .then(({data}) => {
        this.todos = data;
      })
      .catch(() => {
        this.$refs.errorMessage.show('Unable to load todos');
      });
  },
  methods: {
    handleSubmit() {
      createTodo(this.title)
        .then(({data}) => {
          this.todos = [...this.todos, data];
          this.title = '';
        });
    },
    updateTodo({ id, title, completed }) {
      updateTodo({ id, title, completed })
        .then(({data}) => {
          this.todos = this.todos.map(
            todo => todo.id !== id ? todo : data
          );
        })
    },
    deleteTodo(todoId) {
      deleteTodo(todoId)
        .then(() => {
          this.todos = this.todos.filter(todo => todo.id !== todoId);
        })
    }
  }
}
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button type="button" class="todoapp__toggle-all" :class="{ active: activeTodos.length === 0 }" />

        <form @submit.prevent="handleSubmit">
          <input type="text" class="todoapp__new-todo" placeholder="What needs to be done?" v-model="title" />
        </form>
      </header>

      <TransitionGroup name="list" tag="section" class="todoapp__main">
        <TodoItem 
          v-for="todo, index of visibleTodos" 
          v-bind:key="todo.id"
          :todo="todo"
          @update="updateTodo"
          @delete="deleteTodo(todo.id)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count">
          {{ activeTodos.length }} items left
        </span>

        <StatusFilter 
          v-model="status"
        />

        <button type="button" class="todoapp__clear-completed" v-if="activeTodos.length > 0">
          Clear completed
        </button>
      </footer>
    </div>

    <Message 
      class="is-warning" 
      ref="errorMessage"
    >
      <template #default="{text}">
        <p>{{text}}</p>
        <button class="delete" @click="$refs.errorMessage.hide()">x</button>
      </template>

      <template #header>
			  <p>Server Error</p>
      </template>
    </Message>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  max-height: 0;
  transform: scaleY(0);
}
</style>
