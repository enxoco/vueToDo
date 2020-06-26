<template>
  <div class="container">
    <div>
      <h1 class="title">Time Tracker</h1>
      <div class="flex">
        <img src="logo.svg" height="500px" />

        <ul>
          <li v-for="(task, index) in tasks" :data-index="index">
            <input v-model="task.item" @keyup="saveTasks()" />
            <a @click="deleteTask(index)">
              <x-circle-icon size="1.5x" class="custom-class inactive"></x-circle-icon>
            </a>
            <a @click="start(index)" disabled="(task.active) ? 'disabled' : ''">
              <play-circle-icon
                size="1.5x"
                class="custom-class active"
                :class="{ active: task.active }"
              ></play-circle-icon>
            </a>
            <a @click="stop(index)">
              <stop-circle-icon size="1.5x" class="custom-class" :class="{ inactive: task.active }"></stop-circle-icon>
            </a>
            {{ (formattedElapsedTime(index)) ? formattedElapsedTime(index) : "00:00:00" }}
          </li>
          <div class="links">
            <a @click="addTask">
              <img src="plus-circle.svg" />
            </a>
          </div>

          Total Time: {{ getTotalTime() }}
        </ul>
      </div>
    </div>
  </div>
</template>
<style scoped>
h1 {
  font-family: "Krona One";
  font-size: 5.5rem;
}
.active {
  stroke: #bfb48f;
}
.inactive {
  stroke: #904e55;
}
ul {
  list-style: none;
}
input {
    border: 1px solid gainsboro;
    padding: 10px;
    box-shadow: 1px 1px 1px 1px gainsboro;
    border-radius: 5px;
    margin-right: 10px;
    margin-bottom: 15px;
    min-width: 300px;
}
.flex {
  display: flex;
}
body {
  background: #bfb48f;
}
a {
  stroke: #252627;
}
</style>
<script>
import {
  PlusCircleIcon,
  StopCircleIcon,
  PlayCircleIcon,
  XCircleIcon
} from "vue-feather-icons";
import * as workerTimers from "worker-timers";

export default {
  components: {
    PlusCircleIcon,
    StopCircleIcon,
    PlayCircleIcon,
    XCircleIcon
  },
  data: function() {
    return {
      tasks: [],
      totalTime: 0
    };
  },
  mounted() {
    if (localStorage.getItem("tasks")) {
      try {
        this.tasks = JSON.parse(localStorage.getItem("tasks"));
      } catch (e) {
        localStorage.removeItem("tasks");
      }
    }
  },
  methods: {
    addTask() {
      this.tasks.push({ item: null, timer: 0, elapsedTime: 0, active: false });
      this.saveTasks();
    },
    saveTasks() {
      const parsed = JSON.stringify(this.tasks);
      localStorage.setItem("tasks", parsed);
    },
    getTotalTime() {
      this.totalTime = 0;
      this.tasks.forEach(task => {
        this.totalTime += task.elapsedTime;
      });
      const date = new Date(null);
      date.setSeconds(this.totalTime / 1000);
      const utc = date.toUTCString();
      this.totalTime = utc.substr(utc.indexOf(":") - 2, 8);
      return utc.substr(utc.indexOf(":") - 2, 8);
    },
    formattedElapsedTime(index) {
      if (this.tasks[index].elapsedTime) {
        const date = new Date(null);
        date.setSeconds(this.tasks[index].elapsedTime / 1000);
        const utc = date.toUTCString();
        return utc.substr(utc.indexOf(":") - 2, 8);
      }
    },
    start(index) {
      if (this.tasks[index].active) {
        return;
      }
      this.tasks[index].timer = workerTimers.setInterval(() => {
        this.tasks[index].elapsedTime += 1000;
        this.saveTasks();
      }, 1000);
      this.tasks[index].active = true;
    },
    stop(index) {
      workerTimers.clearInterval(this.tasks[index].timer);
      this.tasks[index].active = false;
      this.saveTasks();
    },
    reset() {
      this.tasks[index].elapsedTime = 0;
      this.tasks[index].active = false;
      this.saveTasks();
    },
    deleteTask(index) {
      this.tasks.splice(this.tasks.index, 1);
      this.saveTasks();
    }
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
