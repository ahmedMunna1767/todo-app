<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const today = computed(() => {
  const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday']
  const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September',
    'October', 'November', 'December'
  ]
  const d = new Date()
  return {
    day: days[d.getDay()],
    date: `${months[d.getMonth()]} ${d.getDate()}`
  }
});

const new_todo = ref("");// new_todo.value
const showComplete = ref(false);
const todoList = ref(JSON.parse(localStorage.getItem('todo_list')) || []);

watch(todoList, (updatedList) => {
  localStorage.setItem('todo_list', JSON.stringify(updatedList));
}, { deep: true });

function getTodos() {
  if (localStorage.getItem('todo_list')) {
    todoList.value = JSON.parse(localStorage.getItem('todo_list'));
  }
};

function addItem() {
  if (new_todo.value !== '') {
    todoList.value.unshift({
      id: todoList.value.length,
      title: new_todo.value,
      done: false,
    });
  }
  new_todo.value = '';
  return true;
};
function deleteItem(item) {
  todoList.value.splice(todoList.value.indexOf(item), 1);
}
function toggleShowComplete() {
  showComplete.value = !showComplete.value;
}
function clearAll() {
  todoList.value = [];
}
const pending = computed(() => todoList.value.filter(item => !item.done));
const completed = computed(() => todoList.value.filter(item => item.done));
const completedPercentage = computed(() => Math.round((completed.value.length / todoList.value.length) * 100));

onMounted(() => {
  getTodos();
});
</script>

<template>
  <section class="todo-wrapper">
    <div class="title-header">
      <h1 class="todo-title">{{ today.day }}, {{ today.date }}</h1>
      <a class="todo-title" href="https://github.com/ahmedMunna1767/todo-app"><img src="../assets/icons8-github.gif"
          alt="Github" title="Go to Src"></a>
    </div>
    <form @keydown.enter.prevent="">
      <input type="text" class="input-todo" v-bind:class="{ active: new_todo }" placeholder="Take the garbage out"
        v-model="new_todo" v-on:keyup.enter="addItem">
      <div class="btn btn-add" v-bind:class="{ active: new_todo }" @click="addItem">+</div>
    </form>


    <div v-if="pending.length > 0">
      <p class="status busy">You have {{ pending.length }} pending item<span v-if="pending.length > 1">s</span></p>
      <transition name="todo-item" mode="out-in">
        <ul class="todo-list" key="list">
          <li v-for="(item, index) in pending" :key="item.title">
            <input class="todo-checkbox" :id="'item_' + item.id" v-model="item.done" type="checkbox">
            <label :for="'item_' + item.id"></label>
            <span class="todo-text">{{ item.title }}</span>
            <span class="delete" @click="deleteItem(item)"></span>
          </li>
        </ul>
      </transition>
    </div>

    <transition name="slide-fade">
      <p class="status free" v-if="!pending.length"><img src="../assets/beer_celebration.svg" alt="celebration">Time to
        chill!
        You have no todos.</p>
    </transition>

    <div v-if="completed.length > 0 && showComplete">
      <p class="status">Completed tasks: {{ completedPercentage }} %</p>
      <transition name="todo-item" mode="out-in">
        <ul class="todo-list" key="list">
          <li v-for="(item, index) in completed" :key="item.title">
            <input class="todo-checkbox" :id="'item_' + item.id" v-model="item.done" type="checkbox">
            <label :for="'item_' + item.id"></label>
            <span class="todo-text">{{ item.title }}</span>
            <span class="delete" @click="deleteItem(item)"></span>
          </li>
        </ul>
      </transition>
    </div>
    <div class="control-buttons">
      <div class="btn btn-secondary" v-if="completed.length > 0" @click="toggleShowComplete"><span
          v-if="!showComplete">Show</span><span v-else>Hide</span> Complete</div>
      <div class="btn btn-secondary" v-if="todoList.length > 0" @click="clearAll">Clear All</div>
    </div>
  </section>
</template>

<style scoped>
img:hover::after {
  content: attr(title);
  position: absolute;
  background-color: black;
  color: white;
  padding: 0.5rem;
  border-radius: 0.25rem;
  font-size: 0.8rem;
  white-space: nowrap;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0.8;
  z-index: 1;
}

.todo-wrapper {
  width: 400px;
  max-width: 100%;
  min-height: 500px;
  margin: 20px auto 40px;
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 40px 20px;
  -webkit-box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.05);
  box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.05);
  background-color: #f4f7fc;
  overflow: hidden;
  position: relative;
}


.title-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

a {
  text-decoration: none;
}

.todo-title {
  text-align: left;
  font-size: 1.2em;
  color: #f65c65;
  font-weight: normal;
}

form {
  overflow: overlay;
}

form label {
  display: block;
  text-align: center;
  font-size: 1.2em;
}

.btn,
input {
  line-height: 2em;
  border-radius: 3px;
  border: 0;
  display: inline-block;
  margin: 15px 0;
  padding: 0.2em 1em;
  font-size: 1em;
}

input[type='text'] {
  border: 1px solid #ddd;
  min-width: 80%;
  transition: all ease-in 0.25s;
}

input:focus {
  outline: none;
  border: 1px solid #a3b1ff;
}

input::placeholder {
  color: rgba(0, 0, 0, 0.3);
  font-style: italic;
}

.btn {
  text-align: center;
  font-weight: bold;
  cursor: pointer;
  border-width: 1px;
  border-style: solid;
}

.btn-add {
  background: #ddd;
  color: #fefefe;
  border-color: #ddd;
  min-width: 17%;
  pointer-events: none;
  transition: all ease-in 0.25s;
  font-size: 2.2em;
  line-height: 0.5em;
  padding: 0.3em 0.3em;
  float: right;
}

.btn-add.active {
  background: #6664ff;
  border-color: #6664ff;
  pointer-events: visible;
}

.btn-add.active:hover {
  background: #4442f6;
  border-color: #4442f6;
}

.btn-add:active {
  transform: scale(0.95);
}

.control-buttons {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
}

.btn-secondary {
  display: inline-block;
  position: relative;
  border: 0;
  padding: 0;
  margin: 0 10px;
}

.btn-secondary:after {
  position: absolute;
  content: '';
  width: 0;
  height: 3px;
  background-color: #f4586e;
  bottom: 0px;
  left: 0;
  transition: all ease-in 0.25s;
}

.btn-secondary:hover:after {
  width: 100%;
}


ul.todo-list {
  padding: 0;
  margin-bottom: 30px;
}

ul.todo-list li {
  position: relative;
  list-style-type: none;
  display: block;
  margin: 10px 0;
  background: #e0e8f5;
  border-radius: 3px;
  padding-left: 38px;
  /* custom checkbox width + 16 */
  padding-top: 12px;
  padding-bottom: 12px;
  padding-right: 49px;
  /* delete button + 5 */
  overflow: hidden;
}

ul.todo-list.archived li {
  background: #fff;
}

.todo-text {
  position: relative;
  display: inline-block;
  padding: 0 0.5em;
}


ul.todo-list li .delete {
  position: absolute;
  height: 100%;
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  cursor: pointer;
  opacity: 0;
  width: 0;
  background-color: #f56468;
  color: #fff;
  transition: all ease-in 0.25s;
}

ul.todo-list li .delete:after {
  position: absolute;
  content: '';
  width: 16px;
  height: 16px;
  top: 50%;
  left: 50%;
  background-image: url('../assets/trash.svg');
  background-repeat: no-repeat;
  background-position: center;
  background-size: contain;
  transform: translate(-50%, -50%) scale(0.5);
  transition: all ease-in 0.25s;
}

ul.todo-list li:hover .delete {
  width: 44px;
  opacity: 1;
}

ul.todo-list li:hover .delete:after {
  transform: translate(-50%, -50%) scale(1);
}

.todo-checkbox {
  position: absolute;
  opacity: 0;
  display: none;
}

.todo-checkbox+label {
  position: absolute;
  cursor: pointer;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 22px;
  height: 22px;
  border-radius: 2px;
  border: 1px solid #cfdcec;
  background-color: #fff;
}

.todo-checkbox:checked+label:after {
  position: absolute;
  content: '';
  top: 30%;
  left: 50%;
  height: 3px;
  width: 6px;
  border: solid #fc6c48;
  border-width: 0 0 2px 2px;
  transform-origin: center center;
  transform: rotate(-45deg) translate(-50%, -50%);
}

.todo-checkbox:checked+label:after {
  display: block;
}

.todo-checkbox:checked~.todo-text {
  color: #888;
  text-decoration: line-through
}

.status.free {
  font-weight: bold;
  text-align: center;
  margin: 40px 0;
}

.status.free img {
  display: block;
  width: 50px;
  margin: 10px auto;
  /* vertical-align: middle; */
}

.todo-item-enter-active,
.todo-item-leave-active {
  transition: opacity ease 0.25s, transform ease-in-out 0.3s;
  transform-origin: left center;
}

/* .todo-item-leave-active below version 2.1.8 */
.todo-item-enter,
.todo-item-leave-to {
  opacity: 0;
  transform: translateX(100%);
}


.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease;
}

/* .slide-fade-leave-active below version 2.1.8 */
.slide-fade-enter,
.slide-fade-leave-to {
  transform: scale(1.1);
  opacity: 0;
}


/* Footer */
footer {
  position: absolute;
  width: 100%;
  text-align: center;
  color: #fff;
  bottom: 20px;
  left: 0;
}

footer a {
  color: #fff;
}
</style>