<template>
  <AppHeader />

  <AppFilters :active-filter="activeFilter" @set-filter="setFilter" />

  <main class="app-main">
    <AppTodoList :todos="filterdTodos" @toggle-todo="toggleTodo" @remove-todo="removeTodo" />

    <AppAddTodo @add-todo="addTodo" />
  </main>

  <AppFooter :stats="stats" />
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import AppHeader from './components/AppHeader.vue';
import AppFilters from './components/AppFilters.vue';
import AppTodoList from './components/AppTodoList.vue';
import AppAddTodo from './components/AppAddTodo.vue';
import AppFooter, { Stats } from './components/AppFooter.vue';
import { Todo } from './types/Todo';
import { Filter } from './types/Filter';

const STORAGE_KEY = 'todos';

interface State {
  todos: Todo[],
  activeFilter: Filter,
}

export default defineComponent({
  components: {
    AppHeader,
    AppFilters,
    AppTodoList,
    AppAddTodo,
    AppFooter,
  },

  data(): State {
    return {
      todos: this.loadTodos(),
      activeFilter: 'All',
    }
  },

  computed: {
    filterdTodos(): Todo[] {
      switch (this.activeFilter) {
        case 'Active':
          return this.activeTodos;
        case 'Done':
          return this.doneTodos;
        case 'All':
        default:
          return this.todos;
      }
    },
    stats(): Stats {
      return {
        active: this.activeTodos.length,
        done: this.doneTodos.length,
      }
    },
    activeTodos(): Todo[] {
      return this.todos.filter(todo => !todo.completed);
    },
    doneTodos(): Todo[] {
      return this.todos.filter(todo => todo.completed);
    }
  },

  methods: {
    loadTodos(): Todo[] {
      const saved = localStorage.getItem(STORAGE_KEY);
      if (saved) {
        try {
          const parsed = JSON.parse(saved) as Todo[];
          if (Array.isArray(parsed) && parsed.length > 0) {
            return parsed;
          }
        } catch (e) {
          console.error('Failed to parse todos from localStorage', e);
        }
      }
      // если нет сохранённых задач или массив пустой
      return [
        { id: 0, text: 'Learn the basics of Vue', completed: true },
        { id: 1, text: 'Learn the basics of Typescript', completed: false },
        { id: 2, text: 'Subscribe to the channel', completed: false },
      ];
    },

    saveTodos() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.todos));
    },

    addTodo(todo: Todo) {
      this.todos.push(todo);
      this.saveTodos();
    },

    toggleTodo(id: number) {
      const targetTodo = this.todos.find((todo: Todo) => todo.id === id);
      if (targetTodo) {
        targetTodo.completed = !targetTodo.completed;
        this.saveTodos();
      }
    },

    removeTodo(id: number) {
      this.todos = this.todos.filter((todo: Todo) => todo.id !== id);
      this.saveTodos();
    },

    setFilter(filter: Filter) {
      this.activeFilter = filter;
    },
  },
})
</script>
