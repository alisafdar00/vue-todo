```vue
<script setup>
import { ref } from 'vue'

const todoInput = ref('')

const todos = ref([])

function addTodo() {
  if (todoInput.value.trim() !== '') {
    todos.value.unshift({
      id: Date.now(),
      text: todoInput.value,
      done: false
    })

    todoInput.value = ''
  }
}

function toggleDone(todo) {
  todo.done = !todo.done
}

function deleteTodo(id) {
  todos.value = todos.value.filter(todo => todo.id !== id)
}
</script>

<template>
  <form class="box" @submit.prevent="addTodo">
    <div class="field">
      <label class="label">ToDo</label>

      <div class="control">
        <input
          type="text"
          class="input"
          v-model="todoInput"
          placeholder="Enter your todo"
        />
      </div>
    </div>

    <button type="submit" class="button is-primary">
      Add ToDo
    </button>
  </form>

  <div
    class="card mb-3"
    v-for="todo in todos"
    :key="todo.id"
  >
    <div class="card-content">
      <div
        class="content"
        :class="{ 'has-text-success': todo.done }"
      >
        {{ todo.text }}
      </div>

      <button
        class="button is-primary mr-3"
        @click="toggleDone(todo)"
      >
        {{ todo.done ? 'Undo' : 'Done' }}
      </button>

      <button
        class="button is-danger"
        @click="deleteTodo(todo.id)"
      >
        Delete
      </button>
    </div>
  </div>
</template>

<style scoped>
.card {
  margin-bottom: 10px;
}
</style>