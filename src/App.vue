<script>
import { computed, reactive, toRefs } from "vue";
import { v4 as uuid } from "uuid";
import Axios from "axios";
import IconCheckCircle from "./components/IconCheckCircle.vue";
import IconCircle from "./components/IconCircle.vue";
import IconDelete from "./components/IconDelete.vue";
import IconEdit from "./components/IconEdit.vue";
import MainHero from "./components/MainHero.vue";
import TabNav from "./components/TabNav.vue";

export default {
  name: "App",
  components: {
    IconCheckCircle,
    IconCircle,
    IconDelete,
    IconEdit,
    MainHero,
    TabNav,
  },

  setup() {
    const token =
      "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC81NC4xNDQuMTU1LjE0NVwvYXBpXC9sb2dpbiIsImlhdCI6MTYzNzAwNDgyNCwiZXhwIjoxNjM3MDA4NDI0LCJuYmYiOjE2MzcwMDQ4MjQsImp0aSI6IlROTTdWNUp4M2tZMWJHdjkiLCJzdWIiOjMyLCJwcnYiOiIyM2JkNWM4OTQ5ZjYwMGFkYjM5ZTcwMWM0MDA4NzJkYjdhNTk3NmY3In0.Esqbu9_CI3tanJR5DcLqYqVtxx_r4Y0u2m7dKMItfG4";
    const state = reactive({
      currentView: "All",
      newTaskInput: "",
      taskList: [],
    });
    Axios.get("http://54.144.155.145/api/items", {
      headers: {
        Authorization: "Bearer " + token,
      },
    }).then((res) => {
      console.log(res);
      console.log(res.data.items.data.length);
      state.taskList = res.data.items.data;
      // console.log("items::::::", state.taskList);
    });

    const taskLists = reactive({
      all: computed(() => state.taskList),
    });

    const taskListOverview = reactive([
      { name: "All", length: computed(() => taskLists.all.length) },
    ]);

    const tasksInView = computed(() => {
      return state.taskList;
    });

    const addTask = () => {
      Axios.post(
        "http://54.144.155.145/api/item",

        {
          id: uuid(),
          title: state.newTaskInput,
          description: state.newTaskInput,
        },
        { headers: { Authorization: "Bearer " + token } }
      ).then((res) => {
        state.taskList.push(res.data.item);
      });
      // state.taskList.push({
      //   id: uuid(),
      //   complete: false,
      //   edit: false,
      //   label: state.newTaskInput,
      // });
      state.newTaskInput = "";
    };

    const toggleEdit = (taskId) => {
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList[taskIndex].edit = !state.taskList[taskIndex].edit;
    };

    const deleteTask = (taskId) => {
      Axios.delete(`http://54.144.155.145/api/item/${taskId}`, {
        headers: { Authorization: "Bearer " + token },
      });
      const taskIndex = state.taskList.findIndex((task) => task.id === taskId);
      state.taskList.splice(taskIndex, 1);
    };

    const setView = (viewLabel) => {
      state.currentView = viewLabel;
    };

    return {
      ...toRefs(state),
      addTask,
      deleteTask,
      setView,
      tasksInView,
      toggleEdit,
      taskListOverview,
    };
  },
};
</script>

<template>
  <main class="main-wrapper">
    <MainHero />
    <div class="new-task-wrapper">
      <input
        type="text"
        placeholder="Type a new todo item"
        class="new-task-input"
        v-model="newTaskInput"
        @keyup.enter="addTask"
      />
      <button class="new-task-button" @click="addTask">+ Add</button>
    </div>
    <TabNav
      :currentView="currentView"
      :taskListOverview="taskListOverview"
      @update-current-view="setView"
    />
    <ul class="task-list">
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
            <IconEdit
              class="task-list-cta-icon"
              @click="toggleEdit(taskItem.id)"
            /><span class="sr-only">Edit</span>
          </p>
          <p>
            <IconDelete
              class="task-list-cta-icon"
              @click="deleteTask(taskItem.id)"
            /><span class="sr-only">Delete</span>
          </p>
        </div>
      </li>
    </ul>
  </main>
</template>

<style>
html {
  background-color: #fbfbfb;
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
