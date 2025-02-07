<script setup lang="ts">
import { ref, computed, watch } from "vue";
import dayjs from "dayjs";

interface Todo {
  text: string;
  completed: boolean;
  isEditing: boolean;
  priority: "high" | "medium" | "low";
  deadline: string | null;
  notified: boolean;
}

const todos = ref<Todo[]>(JSON.parse(localStorage.getItem("todos") || "[]"));
const text = ref("");
const priority = ref<"low" | "medium" | "high">("low");
const deadline = ref<string | null>(null);
const filter = ref("all");
function addTodo() {
  if (text.value.trim() === "") return;
  todos.value.push({
    text: text.value.trim(),
    completed: false,
    isEditing: false,
    priority: priority.value,
    deadline: deadline.value,
    notified: false,
  });
  text.value = "";
  priority.value = "low";
  deadline.value = null;
}

function deleteTodo(index: number) {
  todos.value.splice(index, 1);
}

function toggleTodoStatus(index: number) {
  todos.value[index] = {
    ...todos.value[index],
    completed: !todos.value[index].completed,
  };
}

function editTodo(index: number) {
  todos.value[index].isEditing = true;
}

function saveTodo(index: number, newText: string) {
  todos.value[index].text = newText.trim();
  todos.value[index].isEditing = false;
}

const filteredTodos = computed(() => {
  if (filter.value === "completed") {
    return todos.value.filter((todo) => todo.completed);
  } else if (filter.value === "pending") {
    return todos.value.filter((todo) => !todo.completed);
  }
  return todos.value;
});

const sortedTodos = computed(() => {
  const priorityOrder = { high: 1, medium: 2, low: 3 };
  return [...filteredTodos.value].sort(
    (a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]
  );
});

watch(
  todos,
  (newTodos) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);

function checkReminders() {
  todos.value.forEach((todo) => {
    if (todo.deadline) {
      const timeLeft = dayjs(todo.deadline).diff(dayjs(), "minute");
      if (timeLeft > 0 && timeLeft <= 5 && !todo.notified) {
        alert(`Pengingat: Tugas "${todo.text}" akan segera jatuh tempo!`);
        todo.notified = true;
      }
    }
  });
}
setInterval(checkReminders, 60000); 
</script>

<template>
  <div class="w-full min-h-screen py-6 px-4 bg-gray-50">
    <h1 class="text-3xl md:text-5xl text-center text-sky-500 mb-6">Heavenly Venerable Todo List</h1>

    <!-- Add Todo -->
    <div class="my-3 flex flex-col md:flex-row items-stretch md:items-center justify-center gap-3">
      <input v-model="text" type="text" placeholder="Tambahkan tugas..."
        class="p-2 border rounded-md w-full md:w-auto flex-1" />
      <select v-model="priority" class="p-2 border rounded-md w-full md:w-auto flex-1">
        <option value="low">Rendah</option>
        <option value="medium">Sedang</option>
        <option value="high">Tinggi</option>
      </select>
      <input v-model="deadline" type="datetime-local" class="p-2 border rounded-md w-full md:w-auto flex-1" />
      <button @click="addTodo" class="text-white bg-blue-500 px-4 py-2 rounded-md w-full md:w-auto">
        Tambah
      </button>
    </div>

    <!-- Filter Options -->
    <div class="my-3 flex justify-center gap-3 flex-wrap">
      <button @click="filter = 'all'" :class="filter === 'all' ? 'bg-blue-500 text-white' : 'bg-gray-200'"
        class="px-4 py-2 rounded-md w-full sm:w-auto">
        Semua
      </button>
      <button @click="filter = 'completed'" :class="filter === 'completed' ? 'bg-blue-500 text-white' : 'bg-gray-200'"
        class="px-4 py-2 rounded-md w-full sm:w-auto">
        Selesai
      </button>
      <button @click="filter = 'pending'" :class="filter === 'pending' ? 'bg-blue-500 text-white' : 'bg-gray-200'"
        class="px-4 py-2 rounded-md w-full sm:w-auto">
        Belum Selesai
      </button>
    </div>

    <!-- Todo List -->
    <div class="my-6">
      <ul class="space-y-4">
        <li v-for="(todo, index) in sortedTodos" :key="index"
          class="flex flex-col md:flex-row justify-between items-start md:items-center gap-3 p-4 border rounded-md bg-white shadow-md">
          <div v-if="!todo.isEditing" class="flex w-full justify-between">
            <div class="flex items-center gap-3">
              <input type="checkbox" :checked="todo.completed" @change="toggleTodoStatus(index)" class="w-5 h-5" />
              <div class="flex flex-col">
                <span :class="todo.completed ? 'line-through text-gray-500' : ''" class="text-lg">
                  {{ todo.text }}
                </span>
                <span class="text-sm text-gray-500">
                  Prioritas: {{ todo.priority }}
                  <span v-if="todo.deadline">| Deadline: {{ todo.deadline }}</span>
                </span>
              </div>
            </div>
            <div class="flex gap-2">
              <button @click="editTodo(index)" class="px-3 py-1 bg-yellow-500 text-white rounded-md">
                Edit
              </button>
              <button @click="deleteTodo(index)" class="px-3 py-1 bg-red-500 text-white rounded-md">
                Hapus
              </button>
            </div>
          </div>
          <div v-else>
            <div class="flex flex-wrap items-center justify-center gap-3">
              <input v-model="todo.text" type="text" class="border p-2 rounded-md">
              <button @click="saveTodo(index, todo.text)" class="w-full px-3 py-1 bg-blue-500 text-white rounded-md">
                Simpan
              </button>
            </div>
          </div>
        </li>
      </ul>
    </div>

    <!-- Todo Statistics -->
    <div class="my-6 grid grid-cols-1 sm:grid-cols-3 gap-4 bg-gray-100 p-6 rounded-md shadow-md">
      <div class="text-center">
        <h4 class="text-lg font-bold">Total</h4>
        <p class="text-2xl">{{ todos.length }}</p>
      </div>
      <div class="text-center">
        <h4 class="text-lg font-bold">Selesai</h4>
        <p class="text-2xl">{{ todos.filter(todo => todo.completed).length }}</p>
      </div>
      <div class="text-center">
        <h4 class="text-lg font-bold">Belum Selesai</h4>
        <p class="text-2xl">{{ todos.filter(todo => !todo.completed).length }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
