<template>
  <section class="max-w-4xl mx-auto p-6">
    <h2 class="text-2xl font-bold">Welcome back Betro!</h2>
    <p class="mt-2 text-gray-600 text-sm">
      This is your dashboard. You can view your tasks, create new ones, and
      manage your to-do list effectively. Stay organized and productive with
      Dailo!
    </p>

    <div class="card flex flex-col mt-8 bg-white p-6">
      <div class="flex items-center justify-between">
        <div>
          <h2 class="font-bold text-xl">All Tasks</h2>
          <p class="text-sm text-gray-500">Tasks are listed below</p>
        </div>
        <button class="primary-button" @click="openCreateModal">
          + Add New Task
        </button>
      </div>
      <div class="w-full h-px bg-gray-200 my-4" />

      <!-- Empty State -->
      <div v-if="myTasks.length === 0" class="text-center py-12">
        <p class="text-gray-500">No tasks yet. Create your first task!</p>
      </div>

      <UAlert
        color="neutral"
        title="Heads up!"
        description="You can change the primary color in your app config."
        icon="i-lucide-terminal"
        class="rounded-none"
      />
      <!-- Task List -->
      <div
        v-for="task in myTasks"
        :key="task.id || task.title"
        class="flex items-center w-full justify-between mb-4 p-3 hover:bg-gray-50 transition-colors"
      >
        <div class="flex-1">
          <div
            :class="{
              'border-yellow-500': task.status === TaskStatus.PENDING,
              'border-green-500': task.status === TaskStatus.COMPLETED,
              'border-red-500': task.status === TaskStatus.OVERDUE,
              'border-l-4 pl-2 mb-2': true,
            }"
          >
            <p class="text-xs font-semibold text-gray-500 uppercase">
              {{ task.status }}
            </p>
          </div>
          <h3 class="font-bold text-lg">{{ task.title }}</h3>
          <p class="text-sm text-gray-500 mt-1">
            {{ task.desc }}
          </p>
          <p v-if="task.dueDate" class="text-xs text-gray-400 mt-1">
            Due: {{ formatDate(task.dueDate) }}
          </p>
        </div>
        <div class="flex space-x-2 ml-4">
          <button class="update-button" @click="handleEdit(task)">Edit</button>
          <button class="danger-button" @click="handleDelete(task)">
            Delete
          </button>
        </div>
      </div>
    </div>

    <!-- Create/Edit Modal -->
    <div
      v-if="showModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
      @click.self="closeModal"
    >
      <div class="bg-white p-6 w-full max-w-md">
        <h3 class="text-xl font-bold mb-4">
          {{ isEditing ? "Edit Task" : "Create New Task" }}
        </h3>

        <form @submit.prevent="saveTask">
          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Title
            </label>
            <input
              v-model="formData.title"
              type="text"
              required
              class="w-full px-3 py-2 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Enter task title"
            />
          </div>

          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Description
            </label>
            <textarea
              v-model="formData.desc"
              rows="3"
              class="w-full px-3 py-2 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Enter task description"
            ></textarea>
          </div>

          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Status
            </label>
            <select
              v-model="formData.status"
              class="w-full px-3 py-2 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option :value="TaskStatus.PENDING">Pending</option>
              <option :value="TaskStatus.COMPLETED">Completed</option>
              <option :value="TaskStatus.OVERDUE">Overdue</option>
            </select>
          </div>

          <div class="mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">
              Due Date (Optional)
            </label>
            <input
              v-model="formData.dueDate"
              type="date"
              class="w-full px-3 py-2 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div class="flex justify-end space-x-2">
            <button
              type="button"
              @click="closeModal"
              class="px-4 py-2 text-gray-700 bg-gray-100 hover:bg-gray-200"
            >
              Cancel
            </button>
            <button
              type="submit"
              class="px-4 py-2 text-white bg-blue-600 hover:bg-blue-700"
            >
              {{ isEditing ? "Update" : "Create" }} Task
            </button>
          </div>
        </form>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { TaskStatus } from "@/utils/enums";

// Type definitions
interface Task {
  id?: number;
  title: string;
  desc: string;
  status: TaskStatus;
  dueDate?: string;
  createdAt?: Date;
}

// Initial tasks with IDs
const initialTasks: Task[] = [
  {
    id: 1,
    title: "Go to the gym",
    desc: "Make sure by 4:30, I am at the gym",
    status: TaskStatus.COMPLETED,
    dueDate: "2024-03-15",
    createdAt: new Date(),
  },
  {
    id: 2,
    title: "School Assignment",
    desc: "Do my mathematics assignment",
    status: TaskStatus.PENDING,
    dueDate: "2024-03-20",
    createdAt: new Date(),
  },
  {
    id: 3,
    title: "Overdue Task",
    desc: "This task is overdue",
    status: TaskStatus.OVERDUE,
    dueDate: "2024-03-01",
    createdAt: new Date(),
  },
];

const myTasks = ref<Task[]>(initialTasks);
const showModal = ref(false);
const isEditing = ref(false);
const nextId = ref(4); // For generating new IDs

// Form data for create/edit
const formData = reactive<Partial<Task>>({
  title: "",
  desc: "",
  status: TaskStatus.PENDING,
  dueDate: "",
});

// Helper functions
const formatDate = (date?: string) => {
  if (!date) return "";
  return new Date(date).toLocaleDateString("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
};

const openCreateModal = () => {
  isEditing.value = false;
  Object.assign(formData, {
    title: "",
    desc: "",
    status: TaskStatus.PENDING,
    dueDate: "",
  });
  showModal.value = true;
};

const handleEdit = (task: Task) => {
  isEditing.value = true;
  Object.assign(formData, {
    id: task.id,
    title: task.title,
    desc: task.desc,
    status: task.status,
    dueDate: task.dueDate || "",
  });
  showModal.value = true;
};

const handleDelete = (task: Task) => {
  if (confirm(`Are you sure you want to delete "${task.title}"?`)) {
    myTasks.value = myTasks.value.filter((t) => t.id !== task.id);
    console.log("Task deleted:", task);
  }
};

const saveTask = () => {
  if (!formData.title || !formData.desc) return;

  if (isEditing.value) {
    // Update existing task
    myTasks.value = myTasks.value.map((task) =>
      task.id === formData.id
        ? {
            ...task,
            title: formData.title!,
            desc: formData.desc!,
            status: formData.status as TaskStatus,
            dueDate: formData.dueDate,
          }
        : task,
    );
    console.log("Task updated:", formData);
  } else {
    // Create new task
    const newTask: Task = {
      id: nextId.value++,
      title: formData.title!,
      desc: formData.desc!,
      status: formData.status as TaskStatus,
      dueDate: formData.dueDate,
      createdAt: new Date(),
    };
    myTasks.value.push(newTask);
    console.log("Task created:", newTask);
  }

  closeModal();
};

const closeModal = () => {
  showModal.value = false;
  isEditing.value = false;
  Object.assign(formData, {
    title: "",
    desc: "",
    status: TaskStatus.PENDING,
    dueDate: "",
  });
};
</script>

<style scoped>
/* Optional: Add animations for modal */
.fixed {
  animation: fadeIn 0.2s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
