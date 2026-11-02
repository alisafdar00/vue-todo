```vue
<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
const todoInput = ref('')

const todos = ref([])
const errorMessage = ref('')
const loading = ref('')
/**
 *  Get All Todos
 *  @returns {Promise<void>}
 */

async function fetchTodos() {
  const URL  = "http://127.0.0.1:8000/api/todos"
  axios.get(URL)
    .then(response => {
      todos.value = response.data.map(todo => ({
        id: todo.id,
        text: todo.todo_item,
        done: todo.done,
        created_at: todo.created_at
      }))
      console.log('Fetched todos:', response.data)
    })
    .catch(error => {
      errorMessage.value = 'Error fetching todos'
      console.error('Error fetching todos:', error)
    })
}


/**
 * On mounted, fetch the todos from the API
 */
onMounted(() => {
  fetchTodos()
})

/**
 * Add a new todo item
 * @returns {Promise<void>}
 */
function addTodo() {
  const saveURL = "http://127.0.0.1:8000/api/todos";
  if (todoInput.value.trim() !== '' && todoInput.value.length <= 233) {
    loading.value = true
    axios.post(saveURL, {
      todo_item: todoInput.value,
      done: false
    })
      .then(response => {
        todos.value.unshift({
          id: response.data.data.id,
          text: response.data.data.todo_item,
          done: response.data.data.done,
          created_at: response.data.data.created_at
        })
        // console.log('RESPONSE todos:', response.data.data)
        // console.log('Fetched todos:', todos.value)
        todoInput.value = ''
        loading.value = false
      })
      .catch(error => {
        errorMessage.value = error.response?.data?.message || 'Error adding todo'
        console.error('Error adding todo:', error)
        loading.value = false
      })
  }else if(todoInput.value.length > 233) {
    errorMessage.value = 'Todo cannot exceed 233 characters'
  }else {
    errorMessage.value = 'Todo cannot be empty'
  }
}


/**
 * 
 * @param todo 
 */

function markAsDone(todo) {
  const updateURL = `http://127.0.0.1:8000/api/todos/${todo.id}/done`
  axios.put(updateURL, {
    done: true
  })
    .then(response => {
      todo.done = true
    })
    .catch(error => {
      errorMessage.value = 'Error marking todo as done'
      console.error('Error marking todo as done:', error)
    })
}

function toggleDone(todo) {
  todo.done = !todo.done
}
/**
 * 
 * @param todo 
 */

function unDone(todo) {
  const updateURL = `http://127.0.0.1:8000/api/todos/${todo.id}/undone`
  axios.put(updateURL, {
    done: false
  })
    .then(response => {
      todo.done = false
    })
    .catch(error => {
      errorMessage.value = 'Error marking todo as not done'
      console.error('Error marking todo as not done:', error)
    })
}

/**
 * DELETE a todo item
 * @param id 
 */
function deleteTodo(id) {
  const conftirmDelete = confirm('Are you sure you want to delete this todo?')
  if (!conftirmDelete) {
    return
  }
  const deleteURL = `http://127.0.0.1:8000/api/todos/${id}`
  axios.delete(deleteURL)
    .then(() => {
      todos.value = todos.value.filter(todo => todo.id !== id)
    })
    .catch(error => {
      errorMessage.value = 'Error deleting todo'
      console.error('Error deleting todo:', error)
    })
}

</script>

<template>
  <form class="box" @submit.prevent="addTodo">
    <div class="field">
      <label class="label">ToDo</label>
      <div class="control">
        <textarea v-model="todoInput" class="textarea is-hovered shadow"  placeholder="Enter your todo"></textarea>
      </div>
      <p class="has-text-danger">{{ todoInput.length }} / 233 Characters should be added</p>
    </div>

    <button type="submit" v-if="!loading" class="button is-primary" :disabled="todoInput.trim() === '' || todoInput.length > 233" >
      Add ToDo
    </button>
    <button type="submit" v-else class="button is-primary is-loading" disabled>
      Adding...
    </button>
  </form>

  <article class="message is-danger" v-if="errorMessage">
    <div class="message-header">
      <p>Error Message: </p>
      <button class="delete" aria-label="delete"></button>
    </div>
    <div class="message-body">
      {{ errorMessage }}
    </div>
  </article>
  <div
    class="card mb-3 shadow p-2"
    v-for="todo in todos"
    :key="todo.id"
  >
    <div class="card-content  is-align-items-center">
      <div
        class="content p-2"
        :class="{ 'has-text-success': todo.done, 'strikethrough': todo.done }"
      >
        {{ todo.text }}
       <br />
        <time class="time_class" datetime="todo.created_at">
          Created: {{ todo.created_at }}
        </time>
      
      </div>

      <button
        class="button is-primary mr-2"
        v-if="!todo.done"
        @click="markAsDone(todo)"
      >
     
        Done
      </button>
      <button
        class="button is-success"
        v-if="todo.done"
        @click="unDone(todo)"
      >
      <span class="icon">
        <i class="fas fa-undo"></i>
      </span>
        Undo
      </button>

      <button
        class="button is-danger ml-2"
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
.strikethrough {
  text-decoration: line-through;
}
.time_class {
  font-size: 0.8rem;
  color: #888;
  float: right;
  margin-top: 5px;
  margin-bottom: 5px;
}
</style>