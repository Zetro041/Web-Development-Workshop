<script setup>
import HelloWorld from "./components/HelloWorld.vue";
import { ref, onMounted } from "vue";
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

const hi = "IDENTITY DESIGNATION: ";
let name = ref("");
let number = ref(0);
let capitalCities = ref([]);
const isTrue = ref(true);

// PocketBase collections
let todos = ref([]);
let isLoadingData = ref(true);
let dataError = ref(null);
 
const API_LINK = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR02";
let prayerTime = ref({});

const API_LINK_JOKE = "https://official-joke-api.appspot.com/random_joke";
let joke = ref({});

// Separate name input for todo items to fix variable conflict
let todoName = ref("");

function increment() {
  number.value++;
  console.log(number);
}

let intervalId = null;

function startInterval() {
  intervalId = setInterval(increment, 1000);
}

function stopInterval() {
  clearInterval(intervalId);
  intervalId = null;
}

function addElement() {
  capitalCities.value.push(name.value);
  name.value = "";
}

function getPrayerTime() {
  fetch(API_LINK)
    .then((result) => result.json())
    .then((data) => {
      prayerTime.value = data.prayerTime[0];
      console.log(prayerTime.value);
    });
}

function getJoke() {
  fetch(API_LINK_JOKE)
    .then((result) => result.json())
    .then((data) => {
      joke.value = data;
      console.log(joke);
    });
}

// Function to fetch all todos from PocketBase
async function fetchTodos() {
  isLoadingData.value = true;
  dataError.value = null;
  
  try {
    const records = await pb.collection('todos').getFullList({
      sort: '-created',
    });
    todos.value = records;
    console.log("Fetched todos:", todos.value);
  } catch (error) {
    console.error("Error fetching todos:", error);
    dataError.value = "Failed to connect to DATASPHERE. Error code: " + error.message;
  } finally {
    isLoadingData.value = false;
  }
}

// Function to add a new todo to PocketBase
async function addTodo() {
  if (!todoName.value.trim()) return;
  
  try {
    const data = {
      item: todoName.value,
      completed: false
    };
    
    await pb.collection('todos').create(data);
    todoName.value = "";
    // Refresh the list after adding
    fetchTodos();
  } catch (error) {
    console.error("Error adding todo:", error);
    dataError.value = "DATASPHERE INJECTION FAILED: " + error.message;
  }
}

// Function to toggle todo completion status
async function toggleTodoStatus(todo) {
  try {
    await pb.collection('todos').update(todo.id, {
      completed: !todo.completed
    });
    // Refresh the list after updating
    fetchTodos();
  } catch (error) {
    console.error("Error updating todo:", error);
  }
}

// Function to delete a todo
async function deleteTodo(id) {
  try {
    await pb.collection('todos').delete(id);
    // Refresh the list after deleting
    fetchTodos();
  } catch (error) {
    console.error("Error deleting todo:", error);
  }
}

onMounted(() => {
  getPrayerTime();
  fetchTodos(); // Fetch todos when component mounts
  console.log("Component mounted, fetching todos..."); // Debug log
})
</script>

<template>
  <div class="min-h-screen bg-black text-cyan-400 font-mono p-4">
    <!-- Header Section with Glitch Effect -->
    <div class="border-b border-cyan-500 pb-4 mb-8">
      <h1 class="text-4xl font-bold tracking-tight glitch-text relative overflow-hidden">
        <span class="animate-pulse text-cyan-400">NΞOMΞRPHO5YS</span>
        <span class="absolute top-0 left-1 text-pink-500 opacity-70">NΞOMΞRPHO5YS</span>
      </h1>
      <p class="text-pink-500 text-xs uppercase tracking-widest mt-1">v2.0.77.alpha</p>
    </div>

    <!-- PocketBase Data Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4 mb-6 relative">
      <div class="absolute top-0 right-0 bg-cyan-900 text-xs px-2 py-1">DATASPHERE.CONNECT</div>
      <h2 class="text-xl font-bold text-cyan-400 mb-4 flex items-center">
        <span>NET//TASKS</span>
        <button @click="fetchTodos" class="ml-2 bg-cyan-900 hover:bg-cyan-800 text-cyan-300 px-2 py-1 rounded text-xs border border-cyan-600 focus:outline-none">
          SYNC
        </button>
      </h2>

      <!-- Debug info -->
      <div class="mb-2 text-xs text-pink-400">
        DATA STATUS: {{ isLoadingData ? 'LOADING' : 'READY' }} | RECORDS: {{ todos.length }}
      </div>

      <!-- Add New Todo - FIXED: using todoName instead of name -->
      <div class="flex mb-4">
        <input 
          type="text" 
          v-model="todoName" 
          @keyup.enter="addTodo"
          class="bg-gray-800 text-cyan-300 border border-cyan-700 px-3 py-2 w-full focus:outline-none focus:ring-1 focus:ring-cyan-500 placeholder-cyan-700" 
          placeholder="NEW_TASK" 
        />
        <button 
          @click="addTodo" 
          class="bg-cyan-900 hover:bg-cyan-800 text-cyan-300 px-4 ml-2 border border-cyan-600 focus:outline-none"
        >
          <span class="text-xs tracking-wider">ADD</span>
        </button>
      </div>

      <!-- Loading State -->
      <div v-if="isLoadingData" class="py-4 text-center">
        <div class="inline-block animate-pulse text-cyan-500">ACCESSING_DATASPHERE...</div>
      </div>

      <!-- Error State -->
      <div v-else-if="dataError" class="border border-pink-700 bg-pink-900 bg-opacity-20 p-3 text-pink-400 text-sm">
        {{ dataError }}
      </div>

      <!-- Todo List - Improved visibility -->
      <div v-else-if="todos.length > 0" class="border border-cyan-900 rounded bg-gray-950 p-2">
        <div class="text-xs text-pink-500 mb-2 flex justify-between">
          <span>TASK_REGISTRY</span>
          <span>COUNT: {{ todos.length }}</span>
        </div>

        <ul class="divide-y divide-cyan-900">
          <li 
            v-for="todo in todos" 
            :key="todo.id" 
            class="py-2 px-1 flex items-center justify-between group hover:bg-gray-900"
          >
            <div class="flex items-center">
              <span 
                @click="toggleTodoStatus(todo)" 
                class="cursor-pointer mr-2 w-4 h-4 border flex-shrink-0"
                :class="todo.completed ? 'bg-cyan-500 border-cyan-400' : 'border-cyan-600'"
              ></span>
              <span 
                class="text-sm flex-grow"
                :class="todo.completed ? 'line-through text-cyan-600' : 'text-cyan-300'"
              >
                {{ todo.item }}
              </span>
            </div>
            
            <button 
              @click="deleteTodo(todo.id)" 
              class="bg-pink-900 hover:bg-pink-800 text-pink-300 px-2 py-0.5 rounded text-xs ml-2 flex-shrink-0"
            >
              DELETE
            </button>
          </li>
        </ul>
      </div>

      <!-- Empty State -->
      <div v-else class="border border-cyan-900 rounded bg-gray-950 p-4 text-gray-400 text-sm italic text-center">
        [NO_TASKS_FOUND]
      </div>
    </div>

    <!-- Counter Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4 mb-6 relative overflow-hidden">
      <div class="absolute top-0 right-0 bg-cyan-900 text-xs px-2 py-1">SYS.COUNTER</div>
      <h2 class="text-3xl font-bold text-cyan-400 mb-4">{{ number }}</h2>
      <div class="flex space-x-2">
        <button @click="startInterval" class="bg-cyan-900 hover:bg-cyan-800 text-cyan-300 px-4 py-1 rounded border border-cyan-600 focus:outline-none focus:ring-2 focus:ring-cyan-400">
          INIT_SEQ
        </button>
        <button @click="stopInterval" class="bg-pink-900 hover:bg-pink-800 text-pink-300 px-4 py-1 rounded border border-pink-600 focus:outline-none focus:ring-2 focus:ring-pink-400">
          TERM_SEQ
        </button>
      </div>
    </div>

    <!-- Identity Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4 mb-6">
      <div class="flex flex-col mb-4">
        <label class="text-xs text-cyan-500 mb-1">{{ hi }}</label>
        <div class="flex">
          <input type="text" v-model="name" class="bg-gray-800 text-cyan-300 border border-cyan-700 px-3 py-2 w-full focus:outline-none focus:ring-1 focus:ring-cyan-500 placeholder-cyan-700" placeholder="ENTER_ID" />
          <button @click="addElement" class="bg-cyan-900 hover:bg-cyan-800 text-cyan-300 px-4 ml-2 border border-cyan-600 focus:outline-none">
            <span class="text-xs tracking-wider">INJECT</span>
          </button>
        </div>
      </div>

      <!-- Data Registry -->
      <div class="bg-gray-800 border border-cyan-900 p-3 rounded">
        <div class="text-xs text-pink-500 mb-2 flex justify-between">
          <span>DATA_REGISTRY</span>
          <span>COUNT: {{ capitalCities.length }}</span>
        </div>
        <ol class="list-decimal pl-5 text-cyan-300">
          <li v-for="city in capitalCities" class="mb-1 text-sm">{{ city }}</li>
        </ol>
        <div v-if="capitalCities.length === 0" class="text-gray-600 text-sm italic">
          [NO_DATA_FOUND]
        </div>
      </div>
    </div>

    <!-- Toggle Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4 mb-6">
      <div class="flex justify-between items-center">
        <h1 v-if="isTrue" class="text-cyan-400 glitch-text">ΞNGLΔND_15_MY_C1TY</h1>
        <button @click="isTrue = !isTrue" class="bg-pink-900 hover:bg-pink-800 text-pink-300 px-3 py-1 rounded border border-pink-600 text-xs">
          {{ isTrue ? 'DISABLE' : 'ENABLE' }}
        </button>
      </div>
    </div>

    <!-- Prayer Time Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4 mb-6">
      <div class="flex justify-between items-center mb-3">
        <h2 class="text-cyan-400 font-bold">PRAYER_TIMES</h2>
        <button @click="getPrayerTime" class="bg-cyan-900 hover:bg-cyan-800 text-cyan-300 px-3 py-1 rounded border border-cyan-600 text-xs">
          REFRESH
        </button>
      </div>

      <div class="border border-cyan-900 rounded bg-gray-950 p-2">
        <div class="text-xs text-pink-500 mb-2 uppercase tracking-wider">
          TIME_SYNC
        </div>
        <table class="w-full text-center">
          <thead>
            <tr class="border-b border-cyan-900">
              <th class="py-1 px-2 text-xs">FAJR</th>
              <th class="py-1 px-2 text-xs">DHUHA</th>
              <th class="py-1 px-2 text-xs">DHUHR</th>
              <th class="py-1 px-2 text-xs">ASR</th>
              <th class="py-1 px-2 text-xs">MAGHRIB</th>
              <th class="py-1 px-2 text-xs">ISHA</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.fajr || '00:00' }}</td>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.dhuha || '00:00' }}</td>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.dhuhr || '00:00' }}</td>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.asr || '00:00' }}</td>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.maghrib || '00:00' }}</td>
              <td class="py-2 text-sm text-cyan-400">{{ prayerTime.isha || '00:00' }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Joke Module -->
    <div class="bg-gray-900 border border-cyan-700 rounded-md p-4">
      <div class="flex justify-between items-center mb-3">
        <h2 class="text-cyan-400 font-bold">HUMOR_PROTOCOL</h2>
        <button @click="getJoke" class="bg-pink-900 hover:bg-pink-800 text-pink-300 px-3 py-1 rounded border border-pink-600 text-xs">
          EXECUTE
        </button>
      </div>

      <div v-if="joke.setup" class="border border-pink-900 rounded bg-gray-950 p-4">
        <p class="text-cyan-300 mb-3">{{ joke.setup }}</p>
        <p class="text-pink-500 italic border-t border-pink-900 pt-2">{{ joke.punchline }}</p>
      </div>
      <div v-else class="border border-cyan-900 rounded bg-gray-950 p-4 text-gray-600 italic text-sm">
        [NO_HUMOR_FOUND]
      </div>
    </div>
  </div>
</template>

<style scoped>
.glitch-text {
  text-shadow: 0 0 2px #3bf5ff, 0 0 5px #3bf5ff;
  position: relative;
}

@keyframes glitch {
  0% { transform: translate(0) }
  20% { transform: translate(-2px, 2px) }
  40% { transform: translate(-2px, -2px) }
  60% { transform: translate(2px, 2px) }
  80% { transform: translate(2px, -2px) }
  100% { transform: translate(0) }
}
</style>