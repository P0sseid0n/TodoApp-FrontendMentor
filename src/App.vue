<script setup lang="ts">
import { computed, reactive, ref } from 'vue';

const allTasks = ref([
   {
      id: 1,
      title: 'Lorem ipsum dolor sit amet consectetur adipisicing elit. Quisquam, quod.',
      completed: false,
   },
   {
      id: 2,
      title: 'Tempora, quibusdam.',
      completed: true,
   },
   {
      id: 3,
      title: 'Wisi, quae.',
      completed: false,
   }
])

const editingTask = reactive({
   id: null as number | null,
   title: '',
   completed: false,
})

const filterTasks = ref<'all' | 'active' | 'completed'>('all');

const addTask = () => {
   if (editingTask.id) {
      allTasks.value = allTasks.value.map((task) => {
         if (task.id === editingTask.id) task.title = editingTask.title;

         return task;
      })

   } else {
      allTasks.value.push({
         id: new Date().getTime(),
         title: editingTask.title,
         completed: editingTask.completed,
      })
   }

   editingTask.id = null;
   editingTask.title = '';
   editingTask.completed = false;
}

const removeTask = (id: number) => {
   allTasks.value = allTasks.value.filter((task) => task.id !== id)
}

const editTask = (id: number) => {
   const task = allTasks.value.find((task) => task.id === id);

   if (task) {
      editingTask.id = task.id;
      editingTask.title = task.title;
      editingTask.completed = task.completed;
   }
}

const toggleTask = (id: number | null) => {
   if (typeof id == 'number') {
      allTasks.value = allTasks.value.map((task) => {
         if (task.id === id) {
            task.completed = !task.completed;
         }
         return task;
      })
   } else {
      editingTask.completed = !editingTask.completed;
   }
}

const clearCompleted = () => {
   allTasks.value = allTasks.value.filter((task) => !task.completed);
}

const isDark = ref(false);

const toggleTheme = () => {
   isDark.value = !isDark.value;
}


const todoBackground = computed(() => (isDark.value ? 'hsl(0, 0%, 100%)' : 'hsl(235, 24%, 19%)'))
const fontColor = computed(() => (isDark.value ? 'hsl(235, 19%, 35%)' : 'hsl(234, 39%, 85%)'))

const viewingTasks = computed(() => {
   if (filterTasks.value === 'active') return allTasks.value.filter((task) => !task.completed)
   else if (filterTasks.value === 'completed') return allTasks.value.filter((task) => task.completed)
   else return allTasks.value;
})


</script>

<template>
   <div id="App" :style="{ background: isDark ? 'hsl(0, 0%, 98%)' : 'hsl(235, 21%, 11%)' }">
      <img id="Background" alt="Background" src="./assets/bg-desktop-light.jpg" v-if="isDark">
      <img id="Background" alt="Background" src="./assets/bg-desktop-dark.jpg" v-else>

      <section>
         <header>
            <h1>TODO</h1>
            <button class="toggleTheme" @click="toggleTheme">
               <img alt="Toggle to dark mode" class="logo" src="./assets/icon-moon.svg" v-if="isDark" />
               <img alt="Toggle to light mode " class="logo" src="./assets/icon-sun.svg" v-else />
            </button>
         </header>
         <div id="Input">
            <button @click="toggleTask(null)" :class="{ checkbox: true, 'checkbox-checked': editingTask.completed }">
               <img alt="Completed task" class="logo" src="./assets/icon-check.svg" v-if="editingTask.completed" />
            </button>
            <input type="text" placeholder="Create a new todo..." @keypress.enter="addTask" v-model="editingTask.title" />
         </div>
         <main>
            <div @click.self="editTask(task.id)" class="todo" v-for="task in viewingTasks">
               <button @click="toggleTask(task.id)" :class="{ checkbox: true, 'checkbox-checked': task.completed }">
                  <img alt="Completed task" class="logo" src="./assets/icon-check.svg" v-if="task.completed" />
               </button>
               <h3>
                  {{ task.title }}
               </h3>
               <button class="delete" @click="removeTask(task.id)">
                  <img alt="Remove task" class="logo" src="./assets/icon-cross.svg" />
               </button>
            </div>
         </main>
         <footer>
            <p>{{ viewingTasks.filter(task => !task.completed).length }} items left</p>
            <div>
               <button @click="filterTasks = 'all'"
                  :class="{ filter: true, 'filter-active': filterTasks === 'all' }">All</button>
               <button @click="filterTasks = 'active'"
                  :class="{ filter: true, 'filter-active': filterTasks === 'active' }">Active</button>
               <button @click="filterTasks = 'completed'"
                  :class="{ filter: true, 'filter-active': filterTasks === 'completed' }">Completed</button>
            </div>
            <button class="clearCompleted" @click="clearCompleted">
               Clear Completed
            </button>
         </footer>

         <p>Drag and drop to reorder list</p>
      </section>
   </div>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Josefin+Sans:wght@400;700&display=swap');

* {
   margin: 0;
   padding: 0;
   box-sizing: border-box;
   color: v-bind(fontColor);

   font-family: 'Josefin Sans', sans-serif;
}

#App {
   height: 100vh;
   display: flex;
   justify-content: center;
}

#Background {
   position: absolute;
   width: 100%;
}

section {
   position: relative;
   z-index: 10;
   max-width: 768px;
   width: 100%;
   height: 100%;
   display: flex;
   flex-direction: column;
   justify-content: center;
   align-items: center;

   text-align: center;

   padding: 64px 0;
}

header {
   display: flex;
   justify-content: space-between;
   align-items: center;
   width: 100%;
}

header h1 {
   font-size: 2.5rem;
   font-weight: 700;
   letter-spacing: 0.5rem;
   color: hsl(0, 0%, 100%);
}

header button {
   background: none;
   border: none;
   cursor: pointer;
}

#Input {
   background: v-bind(todoBackground);
   border-radius: 10px;
   width: 100%;
   margin-top: 48px;
   display: flex;
   align-items: center;
   padding-left: 24px;
}

#Input input {
   width: 100%;
   height: 64px;
   border: none;
   background: none;
   font-size: 1rem;
   font-weight: 700;
   padding: 0 16px;
   outline: none;
}

.checkbox {
   width: 24px;
   height: 24px;
   border-radius: 50%;
   background: transparent;
   margin: 20px 16px 20px 0;
   cursor: pointer;
   display: flex;
   justify-content: center;
   align-items: center;
   border: 2px solid hsl(237, 14%, 26%);


   &.checkbox-checked {
      background: linear-gradient(hsl(192, 100%, 67%), hsl(280, 87%, 65%));
      border: none;
   }

   &:hover:not(.checkbox-checked) {
      border: 2px solid transparent;
      background: linear-gradient(hsl(192, 100%, 67%), hsl(280, 87%, 65%)) border-box;
      -webkit-mask: linear-gradient(#fff 0 0) padding-box,
         linear-gradient(#fff 0 0);
      -webkit-mask-composite: xor;
      mask-composite: exclude;

      transition: .1s;
   }
}



main {
   width: 100%;
   margin-top: 40px;
   background: v-bind(todoBackground);
   border-radius: 10px 10px 0 0;
   height: 480px;
   overflow-y: auto;
}

.todo {
   display: flex;
   align-items: center;
   padding: 8px 24px;
   border-bottom: 1px solid hsl(237, 14%, 26%);

   &:hover .delete {
      opacity: 1;
      display: block;
   }
}

.todo h3 {
   font-size: 1rem;
   font-weight: 700;
   margin-left: 16px;
}

.todo button.delete {
   background: none;
   border: none;
   cursor: pointer;
   margin-left: auto;
   display: none;
   opacity: 0;
   transition: opacity .1s;
}


footer {
   display: flex;
   justify-content: space-between;
   align-items: center;
   width: 100%;
   padding: 16px 24px;
   border-top: 1px solid hsl(237, 14%, 26%);
   background: hsl(235, 24%, 19%);
   background: v-bind(todoBackground);

   border-radius: 0 0 10px 10px;

   * {
      color: hsl(233, 14%, 35%);
   }

   *:hover {
      color: v-bind(fontColor);
   }

   p {
      font-size: 0.9rem;
      font-weight: 700;
   }

   button {
      background: none;
      border: none;
      cursor: pointer;
      font-size: 1rem;
      font-weight: 700;
      opacity: 0.5;
   }

   &+p {
      text-align: center;
      font-size: 0.8rem;
      font-weight: 700;
      color: hsl(233, 14%, 35%);
      margin-top: 64px;
   }
}

.filter {
   margin: 0 8px;
}

.filter-active {
   color: hsl(220, 98%, 61%);
}
</style>
