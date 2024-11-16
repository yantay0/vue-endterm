<template>
  <q-page padding>
    <q-toast ref="toast" position="top-right" />

    <div class="task-input">
      <q-input v-model="newTask.title" label="Task Title" outlined />
      <q-select v-model="newTask.priority" :options="['low', 'medium', 'high']" label="Priority" outlined />
      <q-btn label="Add Task" color="primary" @click="addTask" />

      <q-badge color="primary" floating>
        Total Tasks: {{ totalTasks }}
      </q-badge>
    </div>

    <div class="task-stats">
      <q-badge color="green">
        Pending Tasks: {{ pendingTasks }}
      </q-badge>
      <q-badge color="grey">
        Completed Tasks: {{ completedTasks }}
      </q-badge>
    </div>

    <transition-group name="fade" tag="ul">
      <li v-for="task in sortedTasks" :key="task.id" :id="'task-' + task.id" :class="{ completed: task.completed }">
        <q-checkbox v-model="task.completed" @update:model-value="updateTaskCompletion(task)" />
        <span :class="{
          'high-priority': task.priority === 'high',
          'medium-priority': task.priority === 'medium'
        }">
          {{ task.title }}
        </span>
        <q-btn icon="edit" flat @click="enableEdit(task.id)" />
        <q-btn icon="delete" flat @click="deleteTask(task.id)" />
      </li>
    </transition-group>
  </q-page>
</template>


<script lang="ts">
import { defineComponent, reactive, ref, computed, nextTick, watch } from 'vue';
import { Task } from '../types/task';
import { useQuasar } from 'quasar';

export default defineComponent({
  name: 'IndexPage',
  setup() {
    const $q = useQuasar();
    const tasks = reactive<Task[]>([]);
    const newTask = reactive<Task>({
      id: 0,
      title: '',
      priority: 'low',
      completed: false,
    });
    const editTaskId = ref<number | null>(null);

    const addTask = async () => {
      if (!newTask.title.trim()) {
        $q.notify({
          message: 'Task title cannot be empty',
          type: 'warning',
        });
        return;
      }
      const taskToAdd = { ...newTask, id: Date.now() };
      tasks.push(taskToAdd);
      newTask.title = '';
      $q.notify({
        message: 'Task added!',
        type: 'positive',
      });
      await nextTick();
      document.getElementById(`task-${taskToAdd.id}`)?.scrollIntoView({ behavior: 'smooth' });
    };

    const deleteTask = (id: number) => {
      const index = tasks.findIndex(task => task.id === id);
      if (index !== -1) {
        tasks.splice(index, 1);
        $q.notify({
          message: 'Task deleted!',
          type: 'negative',
        });
      }
    };

    const updateTaskCompletion = (task: Task) => {
      if (task.completed) {
        $q.notify({
          message: 'Task completed!',
          type: 'info',
        });
      }
    };


    const enableEdit = (id: number) => {
      editTaskId.value = id;
    };

    const saveTask = (id: number) => {
      if (editTaskId.value === id) {
        editTaskId.value = null;
        $q.notify({
          message: 'Task updated!',
          type: 'info',
        });
      }
    };

    const sortedTasks = computed(() =>
      [...tasks].sort((a, b) => b.priority.localeCompare(a.priority))
    );

    const totalTasks = computed(() => tasks.length);
    const pendingTasks = computed(() => tasks.filter(task => !task.completed).length);
    const completedTasks = computed(() => tasks.filter(task => task.completed).length);

    watch(tasks, (newTasks, oldTasks) => {
      newTasks.forEach((task, index) => {
        if (!oldTasks[index]?.completed && task.completed) {
          $q.notify({
            message: `Task "${task.title}" completed!`,
            type: 'positive',
          });
        }
      });

      if (newTasks.length > oldTasks.length) {
        $q.notify({
          message: 'Task added!',
          type: 'positive',
        });
      } else if (newTasks.length < oldTasks.length) {
        $q.notify({
          message: 'Task removed!',
          type: 'negative',
        });
      }
    }, { deep: true });

    return {
      tasks,
      newTask,
      addTask,
      deleteTask,
      enableEdit,
      saveTask,
      updateTaskCompletion,
      editTaskId,
      sortedTasks,
      totalTasks,
      pendingTasks,
      completedTasks,
    };
  },
});
</script>

<style scoped>
.task-input {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  align-items: center;
}

.q-select {
  min-width: 120px;
}

.task-stats {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  justify-content: flex-start;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.high-priority {
  color: red;
  font-weight: bold;
}

.medium-priority {
  color: orange;
  font-weight: bold;
}

.completed {
  text-decoration: line-through;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
