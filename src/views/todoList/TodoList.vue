<script>
import { computed, reactive, toRefs } from "vue";
import Axios from "axios";

import IconDelete from "../IconDelete.vue";
import Navbar from "@/component/navbar.vue";

export default {
  name: "App",
  components: {
    IconDelete,
    Navbar,
  },

  mounted() {
    console.log("token::::", this.token);
    this.token = localStorage.getItem("token");
  },

  setup() {
    const state = reactive({
      token: localStorage.getItem("token"),
      taskList: [],
    });
    if (state.token != "") {
      Axios.get("http://54.144.155.145/api/items", {
        headers: {
          Authorization: "Bearer " + state.token,
        },
      }).then((res) => {
        console.log(res);
        state.taskList = res.data.items.data;
      });
    }
    const taskLists = reactive({
      all: computed(() => state.taskList),
    });

    const tasksInView = computed(() => {
      return state.taskList;
    });

    const toggleEdit = (taskId) => {
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList[taskIndex].edit = !state.taskList[taskIndex].edit;
    };

    const deleteTask = (taskId) => {
      Axios.delete(`http://54.144.155.145/api/item/${taskId}`, {
        headers: { Authorization: "Bearer " + state.token },
      });
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList.splice(taskIndex, 1);
    };

    return {
      ...toRefs(state),
      deleteTask,
      tasksInView,
      toggleEdit,
    };
  },
  methods: {
    addtodo() {
      this.$router.push("/createtodo");
    },
  },
};
</script>

<template>
  <Navbar />
  <div v-if="token == null || token == ''" id="home">
    <h2>Welcome to the Todo list App</h2>
  </div>

  <main v-if="token != null && token != ''" class="main-wrapper">
    <ul class="task-list">
      <button @click="addtodo">Add Todo</button>
      <p v-if="tasksInView.length <= 0">loading...</p>
      <li
        v-for="taskItem in tasksInView"
        :key="taskItem.id"
        class="task-list-item"
      >
        <div class="task-list-checkbox-wrapper"></div>
        <input
          v-if="taskItem.edit"
          class="task-list-edit-input"
          type="text"
          v-model="taskItem.title"
        />
        <p
          v-else
          class="task-list-text"
          :class="taskItem.complete ? 'is-complete' : ''"
        >
          {{ taskItem.title }}
        </p>
        <div class="task-list-cta">
          <p>
            <IconDelete
              class="task-list-cta-icon"
              @click="deleteTask(taskItem.id)"
            />
          </p>
        </div>
      </li>
    </ul>
  </main>
</template>

<style scoped>
html {
  background-color: #fbfbfb;
}
#nav1 {
  display: none;
}
.task-list-checkbox-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.task-list-checkbox {
  position: absolute;
  left: -3px;
  bottom: 2px;
  opacity: 0;
}

.task-list {
  padding: 0;
}

.task-list-cta-icon {
  opacity: 0;
  transition: 0.2s opacity ease-in;
}

.task-list-cta-icon .icon-object {
  fill: #2d2d2d;
}

.task-list-cta-icon:hover .icon-object {
  fill: #0728bf;
}

.task-list-item {
  border: 1px solid #f6f6f6;
  box-shadow: 2px 2px 8px 0 #cfcfcf;
  border-radius: 8px;
  display: flex;
  align-items: center;
  padding: 0 16px;
  margin-bottom: 16px;
}

.task-list-item:hover {
  border: 1px solid #0631f8;
}

.task-list-item:hover .task-list-cta-icon,
.task-list-item:focus .task-list-cta-icon {
  opacity: 1;
}

.task-list-cta {
  display: flex;
  column-gap: 16px;
}

.task-list-edit-input,
.task-list-text {
  margin-left: 12px;
  font-weight: bold;
  flex: 1;
  border: 0;
  font-size: 16px;
}

.task-list-text.is-complete {
  color: #6b6b6b;
  text-decoration: line-through;
}

.new-task-wrapper {
  display: flex;
}

.new-task-input {
  padding: 16px;
  font-weight: 600;
  color: #2d2d2d;
  flex: 1;
  border-top-left-radius: 8px;
  border-bottom-left-radius: 8px;
  border: 1px solid #f6f6f6;
  box-shadow: 2px 2px 8px 0 #c0c0c0;
  letter-spacing: 1px;
  font-size: 1rem;
}

.new-task-input:hover {
  border: 1px solid #0631f8;
}

.new-task-input::placeholder {
  color: #959595;
}

.new-task-button {
  background-color: #0631f8;
  color: #fff;
  padding: 18px 24px;
  font-weight: 900;
  border: 0;
  border-top-right-radius: 8px;
  border-bottom-right-radius: 8px;
  transition: 0.2s background ease-in;
  font-size: 1rem;
}

.new-task-button:hover {
  background-color: #082ac9;
}

.main-wrapper {
  max-width: 600px;
  margin: 0 auto;
}
</style>
