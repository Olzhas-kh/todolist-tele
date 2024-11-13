<template>
  <div>
    <h2 class="title">Todo List</h2>
    
    <div class="filters">
      <input v-model="filter.title" type="text" placeholder="Search by title" />
      <input v-model="filter.createdDate" type="date" placeholder="Filter by date" />
    </div>

    <div class="todo-group">
      <div class="group" v-for="status in ['true', 'false']" :key="status">
        <h3>{{ status === 'true' ? 'Completed' : 'Pending' }}</h3>
        <ul>
          <li v-for="todo in filteredTodos(status)" :key="todo.id" :style="{ color: todo.status ? 'green' : 'black' }">
            <span>{{ todo.title }} - {{ todo.createdDate }}</span>
            <button @click="editTodo(todo)">Edit</button>
            <button @click="deleteTodo(todo.id)">Delete</button>
          </li>
        </ul>
      </div>
    </div>

    <p v-if="todos.length === 0">No todos found.</p>

    <h3>{{ editMode ? 'Edit Todo' : 'Add a New Todo' }}</h3>
    <form @submit.prevent="editMode ? updateTodo() : addTodo()">
      <div>
        <label for="title">Title:</label>
        <input v-model="newTodo.title" type="text" id="title" required />
      </div>
      <div>
        <label for="createdDate">Creation Date:</label>
        <input v-model="newTodo.createdDate" type="date" id="createdDate" required />
      </div>
      <div>
        <label for="status">Status:</label>
        <input v-model="newTodo.status" type="checkbox" id="status" />
      </div>
      <button type="submit">{{ editMode ? 'Update Todo' : 'Add Todo' }}</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      todos: [],
      filter: {
        title: '',
        createdDate: ''
      },
      newTodo: {
        id: null,
        title: '',
        createdDate: '',
        status: false,
      },
      editMode: false, 
    };
  },
  computed: {
    filteredTodos() {
      return status => {
        return this.todos.filter(todo => {
          const matchesStatus = todo.status.toString() === status;
          const matchesTitle = todo.title.toLowerCase().includes(this.filter.title.toLowerCase());
          const matchesDate = this.filter.createdDate ? todo.createdDate === this.filter.createdDate : true;

          return matchesStatus && matchesTitle && matchesDate;
        });
      };
    }
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await axios.get('http://localhost:8080/api/todos');
        this.todos = response.data.content || response.data;
      } catch (error) {
        console.error('Error fetching todos:', error);
      }
    },
    async addTodo() {
      try {
        const response = await axios.post('http://localhost:8080/api/todos', {
          title: this.newTodo.title,
          createdDate: this.newTodo.createdDate,
          status: this.newTodo.status
        });
        this.todos.push(response.data); 
        this.resetForm(); 
      } catch (error) {
        console.error('Error adding todo:', error);
      }
    },
    async updateTodo() {
      try {
        const response = await axios.put(`http://localhost:8080/api/todos/${this.newTodo.id}`, {
          title: this.newTodo.title,
          createdDate: this.newTodo.createdDate,
          status: this.newTodo.status
        });
        const index = this.todos.findIndex(todo => todo.id === this.newTodo.id);
        this.todos[index] = response.data; 
        this.resetForm(); 
      } catch (error) {
        console.error('Error updating todo:', error);
      }
    },
    async deleteTodo(id) {
      try {
        await axios.delete(`http://localhost:8080/api/todos/${id}`);
        this.todos = this.todos.filter(todo => todo.id !== id); 
      } catch (error) {
        console.error('Error deleting todo:', error);
      }
    },
    editTodo(todo) {
      this.newTodo = { ...todo }; 
      this.editMode = true; 
    },
    resetForm() {
      this.newTodo = { id: null, title: '', createdDate: '', status: false };
      this.editMode = false; 
    }
  },
  mounted() {
    this.fetchTodos();
  }
};
</script>

<style scoped>
/* Add styles for the layout and improve the form */
.title {
  text-align: center;
  font-size: 2rem;
  color: #333;
}

.filters {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.filters input {
  padding: 8px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.todo-group {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}

.group {
  width: 45%;
}

.group h3 {
  background-color: #f3f3f3;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 15px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  padding: 10px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
}

button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>
