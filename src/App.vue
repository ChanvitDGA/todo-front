<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import Swal from "sweetalert2";

const apiUrl = "http://13.229.52.109:3000/todos";
const inputValue = ref("");
const todos = ref([]);

const editTask = async (todo) => {
  const { value: newTitle } = await Swal.fire({
    title: "Edit your task",
    input: "text",
    inputValue: "",
    showCancelButton: true,
    confirmButtonText: "Update Task",
    cancelButtonText: "Cancel",
    inputValidator: (value) => {
      if (!value) {
        return "You need to enter a task!";
      }
    },
  });

  if (newTitle) {
    try {
      const { data } = await axios.put(`${apiUrl}/${todo._id}`, {
        ...todo,
        title: newTitle,
      });
      const index = todos.value.findIndex((t) => t._id === todo._id);
      todos.value[index] = data;
    } catch (err) {
      console.error("Error updating todo:", err);
    }
    Swal.fire({
      title: "Task updated successfully!",
      icon: "success",
      timer: 1500,
      showConfirmButton: false,
    });
  }
};

const toggleTask = async (todo) => {
  try {
    const { data } = await axios.put(`${apiUrl}/${todo._id}`, {
      ...todo,
      completed: !todo.completed,
    });
    const index = todos.value.findIndex((t) => t._id === todo._id);
    todos.value[index] = data;
  } catch (err) {
    console.error("Error toggling todo:", err);
  }
};

const addTask = async () => {
  try {
    const { data } = await axios.post(`${apiUrl}`, {
      title: inputValue.value,
      completed: false,
    });
    todos.value.push(data);
  } catch (err) {
    console.error("Error adding todo:", err);
  }

  inputValue.value = "";
};

const deleteTask = async (todo) => {
  try {
    await axios.delete(`${apiUrl}/${todo._id}`);
    todos.value = todos.value.filter((t) => t._id !== todo._id);
  } catch (err) {
    console.error("Error deleting todo:", err);
  }
};

const deleteTaskConfirmation = (todo) => {
  Swal.fire({
    title: "Are you sure?",
    text: "You won't be able to revert this!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#3085d6",
    cancelButtonColor: "#d33",
    confirmButtonText: "Yes, delete it!",
  }).then((result) => {
    if (result.isConfirmed) {
      deleteTask(todo);
      Swal.fire("Deleted!", "Your task has been deleted.", "success");
    }
  });
};

onMounted(async () => {
  try {
    console.log(apiUrl);
    const res = await axios.get(`${apiUrl}`);
    todos.value = res.data;
  } catch (err) {
    console.error("Error fetching todos:", err);
  }
});
</script>

<template>
  <div class="flex flex-col items-center justify-center p-10">
    <div class="flex gap-4 mb-6">
      <input
        type="text"
        placeholder="Add a task..."
        class="input w-80"
        v-model="inputValue"
      />
      <button class="btn btn-primary" @click="addTask">Add Task</button>
    </div>
    <!-- Task Card -->
    <div
      v-for="(todo, index) in todos"
      :key="index"
      class="card bg-[#2a2f37] shadow-sm mt-4 w-96 card-sm"
    >
      <div class="text-white flex justify-between items-center p-1 mx-2">
        <button
          class="btn btn-link text-white no-underline"
          :class="{ 'line-through': todo.completed }"
          @click="toggleTask(todo)"
        >
          {{ todo.title }}
        </button>
        <div class="flex gap-x-2">
          <!-- Edit button -->
          <button @click="editTask(todo)" class="btn btn-ghost p-1">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 512 512"
              class="w-4 h-4"
            >
              <path
                fill="#ffffff"
                d="M471.6 21.7c-21.9-21.9-57.3-21.9-79.2 0L362.3 51.7l97.9 97.9 30.1-30.1c21.9-21.9 21.9-57.3 0-79.2L471.6 21.7zm-299.2 220c-6.1 6.1-10.8 13.6-13.5 21.9l-29.6 88.8c-2.9 8.6-.6 18.1 5.8 24.6s15.9 8.7 24.6 5.8l88.8-29.6c8.2-2.7 15.7-7.4 21.9-13.5L437.7 172.3 339.7 74.3 172.4 241.7zM96 64C43 64 0 107 0 160L0 416c0 53 43 96 96 96l256 0c53 0 96-43 96-96l0-96c0-17.7-14.3-32-32-32s-32 14.3-32 32l0 96c0 17.7-14.3 32-32 32L96 448c-17.7 0-32-14.3-32-32l0-256c0-17.7 14.3-32 32-32l96 0c17.7 0 32-14.3 32-32s-14.3-32-32-32L96 64z"
              />
            </svg>
          </button>
          <!-- Delete button -->
          <button
            class="btn btn-ghost p-1"
            @click="deleteTaskConfirmation(todo)"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 448 512"
              class="w-4 h-4"
            >
              <path
                fill="#ffffff"
                d="M135.2 17.7L128 32 32 32C14.3 32 0 46.3 0 64S14.3 96 32 96l384 0c17.7 0 32-14.3 32-32s-14.3-32-32-32l-96 0-7.2-14.3C307.4 6.8 296.3 0 284.2 0L163.8 0c-12.1 0-23.2 6.8-28.6 17.7zM416 128L32 128 53.2 467c1.6 25.3 22.6 45 47.9 45l245.8 0c25.3 0 46.3-19.7 47.9-45L416 128z"
              />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.line-through {
  text-decoration: line-through;
}
</style>
