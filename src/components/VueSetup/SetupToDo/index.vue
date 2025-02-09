<template>
  <div>
    <section class="todoapp">
      <header class="header">
        <h1>Todos Setup</h1>
        <input class="new-todo"
                autofocus autocomplete="off"
                placeholder="What needs to be done?"
                v-model="state.newTodo"
                @keyup.enter="addTodo">
      </header>
      <section class="main" v-show="state.todos.length">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="state.allDone">
        <label for="toggle-all">Mark all as complete</label>
        <ul class="todo-list">
          <li v-for="todo in state.filteredTodos"
              class="todo"
              :key="todo.id"
              :class="{ completed: todo.completed, editing: todo === state.editedTodo }">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input class="edit" type="text"
                    v-model="todo.title"
                    v-todo-focus="todo === state.editedTodo"
                    @blur="doneEdit(todo)"
                    @keyup.enter="doneEdit(todo)"
                    @keyup.escape="cancelEdit(todo)"
            >
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="state.todos.length">
          <span class="todo-count">
            <strong>{{ state.remaining }}</strong>
            <span>{{ state.remainingText }}</span>
          </span>
        <ul class="filters">
          <li>
            <router-link to="/vue-setup/todo/all" active-class="selected">All</router-link>
          </li>
          <li>
            <router-link to="/vue-setup/todo/active" active-class="selected">Active</router-link>
          </li>
          <li>
            <router-link to="/vue-setup/todo/completed" active-class="selected">Completed</router-link>
          </li>
        </ul>

        <button class="clear-completed" @click="removeCompleted" v-show="state.todos.length > state.remaining">
          Clear completed
        </button>
      </footer>
    </section>
  </div>
</template>
<script lang="ts" setup>
import { watch, reactive, computed, watchEffect, onMounted, onUnmounted } from 'vue'
import { useRoute, RouterLink } from 'vue-router';
const STORAGE_KEY = 'todos-vuejs-3.x'
const todoStorage = {
  fetch () {
    const todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    todos.forEach((todo, index) => {
      todo.id = index
    })
    todoStorage.uid = todos.length
    return todos
  },
  save (todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}

const filters = {
  all (todos) {
    return todos
  },
  active (todos) {
    return todos.filter((todo) => {
      return !todo.completed
    })
  },
  completed (todos) {
    return todos.filter(function (todo) {
      return todo.completed
    })
  }
}

const pluralize = (n) => {
  return n === 1 ? 'item' : 'items'
}

const state = reactive({
  todos: todoStorage.fetch(),
  editedTodo: null,
  newTodo: '',
  beforeEditCache: '',
  visibility: 'all',
  remaining: computed(() => {
    return filters.active(state.todos).length
  }),
  remainingText: computed(() => {
    return ` ${pluralize(state.remaining)} left`
  }),
  filteredTodos: computed(() => {
    return filters[state.visibility](state.todos)
  }),
  allDone: computed({
    get: function () {
      return state.remaining === 0
    },
    set: function (value) {
      state.todos.forEach((todo) => {
        todo.completed = value
      })
    }
  })
})

const route = useRoute()
// 自定义指令
const vTodoFocus = (el, { value }) => {
  if (value) {
    el.focus()
  }
}
watchEffect(() => {
  todoStorage.save(state.todos)
})

const onHashChange = (status) => {
  if (filters[status]) {
    state.visibility = status
  } else {
    window.location.hash = ''
    state.visibility = 'all'
  }
}

const addTodo = () => {
  const value = state.newTodo && state.newTodo.trim()
  if (!value) {
    return
  }
  state.todos.push({
    id: todoStorage.uid++,
    title: value,
    completed: false
  })
  state.newTodo = ''
}

const removeTodo = (todo) => {
  state.todos.splice(state.todos.indexOf(todo), 1)
}

const editTodo = (todo) => {
  state.beforeEditCache = todo.title
  state.editedTodo = todo
}

const doneEdit = (todo) => {
  if (!state.editedTodo) {
    return
  }
  state.editedTodo = null
  todo.title = todo.title.trim()
  if (!todo.title) {
    removeTodo(todo)
  }
}

const cancelEdit = (todo) => {
  state.editedTodo = null
  todo.title = state.beforeEditCache
}

const removeCompleted = () => {
  state.todos = filters.active(state.todos)
}

const stopWatch = watch(() => route.params, ({ status }) => {
  onHashChange(status)
}, {
  deep: true,
  immediate: true
})

onMounted(() => {
  // window.addEventListener('hashchange', onHashChange)
  onHashChange(route.params.status)
})

onUnmounted(() => {
  // window.removeEventListener('hashchange', onHashChange)
  stopWatch()
})

defineExpose({
  removeTodo,
  addTodo,
  doneEdit,
  editTodo,
})
</script>
<style>
button {
  margin: 0;
  padding: 0;
  border: 0;
  background: none;
  font-size: 100%;
  vertical-align: baseline;
  font-family: inherit;
  font-weight: inherit;
  color: inherit;
  -webkit-appearance: none;
  appearance: none;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}


.hidden {
  display: none;
}

.todoapp {
  background: #fff;
  margin: 130px 0 40px 0;
  position: relative;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2), 0 25px 50px 0 rgba(0, 0, 0, 0.1);
}

.todoapp input::-webkit-input-placeholder {
  font-style: italic;
  font-weight: 400;
  color: rgba(0, 0, 0, 0.4);
}

.todoapp input::-moz-placeholder {
  font-style: italic;
  font-weight: 400;
  color: rgba(0, 0, 0, 0.4);
}

.todoapp input::input-placeholder {
  font-style: italic;
  font-weight: 400;
  color: rgba(0, 0, 0, 0.4);
}

.todoapp h1 {
  position: absolute;
  top: -140px;
  width: 100%;
  font-size: 80px;
  font-weight: 200;
  text-align: center;
  color: #b83f45;
  -webkit-text-rendering: optimizeLegibility;
  -moz-text-rendering: optimizeLegibility;
  text-rendering: optimizeLegibility;
}

.new-todo,
.edit {
  position: relative;
  margin: 0;
  width: 100%;
  font-size: 24px;
  font-family: inherit;
  font-weight: inherit;
  line-height: 1.4em;
  color: inherit;
  padding: 6px;
  border: 1px solid #999;
  box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
  box-sizing: border-box;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.new-todo {
  padding: 16px 16px 16px 60px;
  height: 65px;
  border: none;
  background: rgba(0, 0, 0, 0.003);
  box-shadow: inset 0 -2px 1px rgba(0, 0, 0, 0.03);
}

.main {
  position: relative;
  z-index: 2;
  border-top: 1px solid #e6e6e6;
}

.toggle-all {
  width: 1px;
  height: 1px;
  border: none; /* Mobile Safari */
  opacity: 0;
  position: absolute;
  right: 100%;
  bottom: 100%;
}

.toggle-all + label {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 45px;
  height: 65px;
  font-size: 0;
  position: absolute;
  top: -65px;
  left: -0;
}

.toggle-all + label:before {
  content: "❯";
  display: inline-block;
  font-size: 22px;
  color: #949494;
  padding: 10px 27px 10px 27px;
  -webkit-transform: rotate(90deg);
  transform: rotate(90deg);
}

.toggle-all:checked + label:before {
  color: #484848;
}

.todo-list {
  margin: 0;
  padding: 0;
  list-style: none;
}

.todo-list li {
  position: relative;
  font-size: 24px;
  border-bottom: 1px solid #ededed;
}

.todo-list li:last-child {
  border-bottom: none;
}

.todo-list li.editing {
  border-bottom: none;
  padding: 0;
}

.todo-list li.editing .edit {
  display: block;
  width: calc(100% - 43px);
  padding: 12px 16px;
  margin: 0 0 0 43px;
}

.todo-list li.editing .view {
  display: none;
}

.todo-list li .toggle {
  text-align: center;
  width: 40px;
  /* auto, since non-WebKit browsers doesn't support input styling */
  height: auto;
  position: absolute;
  top: 0;
  bottom: 0;
  margin: auto 0;
  border: none; /* Mobile Safari */
  -webkit-appearance: none;
  appearance: none;
}

.todo-list li .toggle {
  opacity: 0;
}

.todo-list li .toggle + label {
  /*
		Firefox requires `#` to be escaped - https://bugzilla.mozilla.org/show_bug.cgi?id=922433
		IE and Edge requires *everything* to be escaped to render, so we do that instead of just the `#` - https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/7157459/
	*/
  background-image: url("data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23949494%22%20stroke-width%3D%223%22/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: center left;
}

.todo-list li .toggle:checked + label {
  background-image: url("data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%2359A193%22%20stroke-width%3D%223%22%2F%3E%3Cpath%20fill%3D%22%233EA390%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22%2F%3E%3C%2Fsvg%3E");
}

.todo-list li label {
  word-break: break-all;
  padding: 15px 15px 15px 60px;
  display: block;
  line-height: 1.2;
  transition: color 0.4s;
  font-weight: 400;
  color: #484848;
}

.todo-list li.completed label {
  color: #949494;
  text-decoration: line-through;
}

.todo-list li .destroy {
  display: none;
  position: absolute;
  top: 0;
  right: 10px;
  bottom: 0;
  width: 40px;
  height: 40px;
  margin: auto 0;
  font-size: 30px;
  color: #949494;
  transition: color 0.2s ease-out;
}

.todo-list li .destroy:hover,
.todo-list li .destroy:focus {
  color: #c18585;
}

.todo-list li .destroy:after {
  content: "×";
  display: block;
  height: 100%;
  line-height: 1.1;
}

.todo-list li:hover .destroy {
  display: block;
}

.todo-list li .edit {
  display: none;
}

.todo-list li.editing:last-child {
  margin-bottom: -1px;
}

.footer {
  padding: 10px 15px;
  height: 20px;
  text-align: center;
  font-size: 15px;
  border-top: 1px solid #e6e6e6;
}

.footer:before {
  content: "";
  position: absolute;
  right: 0;
  bottom: 0;
  left: 0;
  height: 50px;
  overflow: hidden;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2), 0 8px 0 -3px #f6f6f6,
    0 9px 1px -3px rgba(0, 0, 0, 0.2), 0 16px 0 -6px #f6f6f6,
    0 17px 2px -6px rgba(0, 0, 0, 0.2);
}

.todo-count {
  float: left;
  text-align: left;
}

.todo-count strong {
  font-weight: 300;
}

.filters {
  margin: 0;
  padding: 0;
  list-style: none;
  position: absolute;
  right: 0;
  left: 0;
}

.filters li {
  display: inline;
}

.filters li a {
  color: inherit;
  margin: 3px;
  padding: 3px 7px;
  text-decoration: none;
  border: 1px solid transparent;
  border-radius: 3px;
}

.filters li a:hover {
  border-color: #db7676;
}

.filters li a.selected {
  border-color: #ce4646;
}

.clear-completed,
html .clear-completed:active {
  float: right;
  position: relative;
  line-height: 19px;
  text-decoration: none;
  cursor: pointer;
}

.clear-completed:hover {
  text-decoration: underline;
}

.info {
  margin: 65px auto 0;
  color: #4d4d4d;
  font-size: 11px;
  text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
  text-align: center;
}

.info p {
  line-height: 1;
}

.info a {
  color: inherit;
  text-decoration: none;
  font-weight: 400;
}

.info a:hover {
  text-decoration: underline;
}

/*
	Hack to remove background from Mobile Safari.
	Can't use it globally since it destroys checkboxes in Firefox
*/
@media screen and (-webkit-min-device-pixel-ratio: 0) {
  .toggle-all,
  .todo-list li .toggle {
    background: none;
  }

  .todo-list li .toggle {
    height: 40px;
  }
}

@media (max-width: 430px) {
  .footer {
    height: 50px;
  }

  .filters {
    bottom: 10px;
  }
}

:focus,
.toggle:focus + label,
.toggle-all:focus + label {
  box-shadow: 0 0 2px 2px #cf7d7d;
  outline: 0;
}
</style>
