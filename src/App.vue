<template>
  <div>
    <header>
      <h1>Aww Todo</h1>

      <form @submit.prevent="submit" class="task-add">
        <input
          class="task-add-box"
          type="text"
          v-model="state.form.task"
          placeholder="Enter task..."
          required
        />
      </form>
    </header>

    <main>
      <div class="card">
        <ul>
          <li
            v-for="task in state.tasks.results"
            :key="task.id"
            @dblclick="markedAsCompleted(task.id, task.title, task.completed)"
          >
            <div class="task-text">
              <textarea
                type="text"
                v-if="state.isTaskEdit == task.id"
                v-model="task.title"
                class="edit-box"
              >
              </textarea>
              <span v-else style="cursor: pointer; user-select: none">
                <span v-if="task.completed">
                  <strike>{{ task.title }}</strike>
                </span>
                <span v-else>
                  {{ task.title }}
                </span>
              </span>
            </div>

            <div>
              <span v-if="state.isTaskEdit != task.id">
                <div>
                  <button class="btn-teal" @click="editTaskToggle(task.id)">
                    Edit
                  </button>
                </div>
                <div>
                  <button class="btn-red" @click="deleteTask(task.id)">
                    Delete
                  </button>
                </div>
              </span>

              <span v-else>
                <div>
                  <button
                    class="btn-blue"
                    @click="updateTask(task.id, task.title)"
                  >
                    Save
                  </button>
                </div>
                <div>
                  <button class="btn-dark" @click="cancelTaskToggle(task.id)">
                    Cancel
                  </button>
                </div>
              </span>
            </div>
          </li>
        </ul>
      </div>

      <div class="prev-next">
        <button
          :class="state.tasks.previous === null ? 'btn-disable' : ''"
          :disabled="state.tasks.previous === null"
          @click="prevPage(state.tasks.previous)"
        >
          Prev
        </button>
        <button
          :class="state.tasks.next === null ? 'btn-disable' : ''"
          :disabled="state.tasks.next === null"
          @click="nextPage(state.tasks.next)"
        >
          Next
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import axios from "axios";
import { onMounted, reactive } from "vue";
export default {
  name: "App",

  setup() {
    const state = reactive({
      tasks: [],
      form: {
        task: "",
      },
      isTaskEdit: "",
    });

    onMounted(() => {
      taskList();
    });

    async function taskList() {
      const { data } = await axios.get(
        "https://awwtodo.herokuapp.com/api/todo/"
      );

      state.tasks = data;
    }

    async function nextPage(page) {
      const { data } = await axios.get(page);
      state.tasks = data;
    }
    async function prevPage(page) {
      const { data } = await axios.get(page);
      state.tasks = data;
    }

    async function submit() {
      await axios.post("https://awwtodo.herokuapp.com/api/todo/", {
        title: state.form.task,
      });

      taskList();
      state.form.task = "";
    }

    async function updateTask(id, title) {
      await axios.put(`https://awwtodo.herokuapp.com/api/todo/${id}/`, {
        title: title,
      });

      state.isTaskEdit = "";
    }

    async function deleteTask(id) {
      if (confirm("Do you really want to delete")) {
        await axios.delete(`https://awwtodo.herokuapp.com/api/todo/${id}/`);
      }
      taskList();
    }

    async function markedAsCompleted(id, title, taskCompleted) {
      console.log("completed ", id);
      if (taskCompleted === false) {
        if (confirm("Do you really want to mark as completed")) {
          await axios.put(`https://awwtodo.herokuapp.com/api/todo/${id}/`, {
            title: title,
            completed: true,
          });
        }
      } else {
        if (confirm("Do you really want to unmark as completed")) {
          await axios.put(`https://awwtodo.herokuapp.com/api/todo/${id}/`, {
            title: title,
            completed: false,
          });
        }
      }
      taskList();
    }

    function editTaskToggle(id) {
      state.isTaskEdit = id;
    }
    function cancelTaskToggle(id) {
      if (state.isTaskEdit == id) {
        state.isTaskEdit = "";
      }
    }

    return {
      state,
      nextPage,
      prevPage,
      submit,
      editTaskToggle,
      cancelTaskToggle,
      updateTask,
      deleteTask,
      markedAsCompleted,
    };
  },
};
</script>


<style lang="scss">
* {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 0;
  padding: 0;
}

body {
  background: #ff006a;
}

header {
  padding-top: 10px;
  padding-bottom: 10px;

  h1 {
    font-size: 42px;
    text-align: center;
    color: #fff;
    margin-bottom: 20px;
  }

  .task-add {
    display: flex;
    justify-content: center;
    padding-left: 30px;
    padding-right: 30px;

    .task-add-box {
      appearance: none;
      background: none;
      outline: none;
      border: none;
      background-color: #f3f3f3;
      box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.15);
      display: block;
      width: 100%;
      max-width: 600px;
      padding: 15px;
      border-radius: 8px;
      color: #313131;
      font-size: 20px;
      transition: 0.4s;

      &::placeholder {
        color: #aaa;
      }
      &:focus,
      &:valid {
        color: #fff;
        background-color: #313131;
        box-shadow: 0px 0px 0px rgba(0, 0, 0, 0.15);
      }
    }
  }
}

main {
  max-width: 600px;
  padding-left: 30px;
  padding-right: 30px;
  margin: 0 auto;
  li {
    background: #fff;
    list-style: none;
    padding: 10px;
    margin-top: 10px;
    box-shadow: 0px 3px 6px rgba(107, 73, 73, 0.15);
    border-radius: 3px;
    display: flex;
    justify-content: space-between;

    .task-text {
      width: 490px;
    }

    .edit-box {
      width: 100%;
      resize: none;
      height: 5em;
      overflow: auto;
      font-size: 16px;
    }
  }

  .prev-next {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
  }
}

button {
  font-size: 14px;
  padding: 5px 10px;
  width: 80px;
  margin: 2px 0px;
  outline: none;
  border: none;
  border-radius: 3px;
  background: #fff;
  box-shadow: 0px 1px 3px rgba(107, 73, 73, 0.15);

  &:hover {
    background: #c7c7c7;
    transition: 0.4s;
  }
}

.btn-teal {
  background: #009494;
  color: #fff;
  &:hover {
    background: #017a7a;
  }
}

.btn-red {
  background: #eb0046;
  color: #fff;
  &:hover {
    background: #da0041;
  }
}
.btn-blue {
  background: #008cff;
  color: #fff;
  &:hover {
    background: #017fe6;
  }
}
.btn-dark {
  background: #1d1d1d;
  color: #fff;
  &:hover {
    background: #000000;
  }
}

.btn-disable {
  background: #bbbbbb;
  &:hover {
    background: #bbbbbb;
  }
}
</style>
