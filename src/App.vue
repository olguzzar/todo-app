<script setup>
import { ref, computed, onMounted } from 'vue'

// State
const newTask = ref('')
const tasks = ref([])
const filter = ref('all') // all, active, completed

// Load tasks from localStorage on mount
onMounted(() => {
  const savedTasks = localStorage.getItem('todo-tasks')
  if (savedTasks) {
    tasks.value = JSON.parse(savedTasks)
  } else {
    // Add sample tasks if no tasks exist
    tasks.value = [
      {
        id: 1,
        text: 'Buy groceries',
        completed: true,
        createdAt: new Date()
      },
      {
        id: 2,
        text: 'Water plants',
        completed: false,
        createdAt: new Date()
      },
      {
        id: 3,
        text: 'Clean the house',
        completed: false,
        createdAt: new Date()
      }
    ]
    // Save sample tasks to localStorage
    saveTasks()
  }
})

// Save tasks to localStorage whenever they change
const saveTasks = () => {
  localStorage.setItem('todo-tasks', JSON.stringify(tasks.value))
}

// Add a new task
const addTask = () => {
  if (newTask.value.trim()) {
    tasks.value.push({
      id: Date.now(),
      text: newTask.value.trim(),
      completed: false,
      createdAt: new Date()
    })
    newTask.value = ''
    saveTasks()
  }
}

// Toggle task completion
const toggleTask = (task) => {
  task.completed = !task.completed
  saveTasks()
}

// Remove a task
const removeTask = (taskId) => {
  tasks.value = tasks.value.filter(task => task.id !== taskId)
  saveTasks()
}

// Clear all completed tasks
const clearCompleted = () => {
  tasks.value = tasks.value.filter(task => !task.completed)
  saveTasks()
}

// Computed tasks based on filter
const filteredTasks = computed(() => {
  switch (filter.value) {
    case 'active':
      return tasks.value.filter(task => !task.completed)
    case 'completed':
      return tasks.value.filter(task => task.completed)
    default:
      return tasks.value
  }
})

// Counts for the footer
const activeCount = computed(() => tasks.value.filter(task => !task.completed).length)
const completedCount = computed(() => tasks.value.filter(task => task.completed).length)
</script>

<template>
  <div class="todo-app">
    <header>
      <h1>My Todo List</h1>
    </header>

    <main>
      <div class="todo-container">
        <!-- Input for new tasks -->
        <div class="task-input">
          <input 
            type="text" 
            v-model="newTask" 
            @keyup.enter="addTask"
            placeholder="What needs to be done?"
            autofocus
          />
          <button @click="addTask" class="add-btn">Add</button>
        </div>

        <!-- Filters -->
        <div class="filters">
          <button 
            :class="{ active: filter === 'all' }" 
            @click="filter = 'all'"
          >
            All
          </button>
          <button 
            :class="{ active: filter === 'active' }" 
            @click="filter = 'active'"
          >
            Active
          </button>
          <button 
            :class="{ active: filter === 'completed' }" 
            @click="filter = 'completed'"
          >
            Completed
          </button>
        </div>

        <!-- Task list -->
        <ul class="task-list" v-if="filteredTasks.length">
          <li v-for="task in filteredTasks" :key="task.id" :class="{ completed: task.completed }">
            <div class="task-content">
              <input 
                type="checkbox" 
                :checked="task.completed" 
                @change="toggleTask(task)"
              />
              <span class="task-text">{{ task.text }}</span>
            </div>
            <button @click="removeTask(task.id)" class="delete-btn">×</button>
          </li>
        </ul>
        <p v-else class="empty-state">No tasks to display</p>

        <!-- Footer with counts and clear completed button -->
        <div class="list-footer" v-if="tasks.length">
          <span>{{ activeCount }} item{{ activeCount !== 1 ? 's' : '' }} left</span>
          <button 
            v-if="completedCount" 
            @click="clearCompleted" 
            class="clear-btn"
          >
            Clear completed
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
:root {
  --primary-color: #6366f1;
  --secondary-color: #818cf8;
  --bg-color: #f9fafb;
  --card-bg: #ffffff;
  --text-color: #1f2937;
  --light-gray: #f3f4f6;
  --border-color: #e5e7eb;
  --success-color: #10b981;
  --danger-color: #ef4444;
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

body {
  font-family: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  background-color: var(--bg-color);
  color: var(--text-color);
  line-height: 1.6;
}
</style>

<style scoped>
.todo-app {
  max-width: 550px;
  margin: 0 auto;
  padding: 2rem 1rem;
  box-sizing: border-box;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 2rem;
}

h1 {
  font-size: 1.75rem;
  font-weight: 600;
  color: var(--primary-color);
  margin: 0;
}

.todo-container {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.task-input {
  display: flex;
  border-bottom: 1px solid var(--border-color);
}

.task-input input {
  flex: 1;
  padding: 1rem;
  font-size: 1rem;
  border: none;
  outline: none;
}

.add-btn {
  background-color: var(--primary-color);
  color: white;
  border: none;
  padding: 0 1.25rem;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s;
}

.add-btn:hover {
  background-color: var(--secondary-color);
}

.filters {
  display: flex;
  padding: 0.75rem;
  background-color: var(--light-gray);
  border-bottom: 1px solid var(--border-color);
}

.filters button {
  background: none;
  border: none;
  font-size: 0.875rem;
  padding: 0.25rem 0.75rem;
  margin-right: 0.5rem;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.filters button:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.filters button.active {
  background-color: var(--primary-color);
  color: white;
}

.task-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.task-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid var(--border-color);
  transition: background-color 0.2s;
}

.task-list li:hover {
  background-color: rgba(0, 0, 0, 0.01);
}

.task-content {
  display: flex;
  align-items: center;
  flex: 1;
}

.task-content input[type="checkbox"] {
  margin-right: 0.75rem;
  height: 18px;
  width: 18px;
  cursor: pointer;
}

.task-text {
  font-size: 0.95rem;
  transition: color 0.2s, text-decoration 0.2s;
}

.completed .task-text {
  color: #868e96;
  text-decoration: line-through;
}

.delete-btn {
  background: none;
  border: none;
  color: var(--danger-color);
  font-size: 1.25rem;
  padding: 0 0.5rem;
  cursor: pointer;
  opacity: 0.6;
  transition: opacity 0.2s;
}

.delete-btn:hover {
  opacity: 1;
}

.list-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1rem;
  color: #868e96;
  font-size: 0.875rem;
}

.clear-btn {
  background: none;
  border: none;
  color: var(--danger-color);
  font-size: 0.875rem;
  cursor: pointer;
  opacity: 0.8;
  transition: opacity 0.2s;
}

.clear-btn:hover {
  opacity: 1;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #868e96;
}

@media (min-width: 1024px) {
  .todo-app {
    padding-top: 3rem;
  }
}
</style>
