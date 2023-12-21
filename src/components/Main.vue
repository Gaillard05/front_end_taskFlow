<script setup lang="ts">
import axios from 'axios';
import { ref, onMounted } from 'vue';

type Task = {
      id: number;
      text: string;
};

const tasksList = ref<Task[]>([]);
const newTask = ref<string | null>(null);
const taskForUpdate = ref<Task | null>(null);
const updatedTask = ref<string | null>(null);

const fetchTasks = async () => {
    try {
        const response = await axios.get('http://localhost:9000/api/task/list-task');
        tasksList.value = response.data;
    } catch (error) {
        console.error('Erreur lors de la récupération des tâches', error);
    }
};

onMounted(fetchTasks);

const addTask = async () => {
  try {
    const response = await axios.post('http://localhost:9000/api/task/add-task', {
      text: newTask.value,
    });

    tasksList.value.push(response.data);
    newTask.value = '';
  } catch (error) {
    console.error('Erreur lors de l\'ajout de la tâche', error);
  }
};

const setTaskForUpdate = (task: Task) => {
  taskForUpdate.value = task;
  updatedTask.value = task.text;
};

const updateTask = async () => {
  try {
    // Stocker la valeur de taskForUpdate.value dans une variable locale
    const taskToUpdate = taskForUpdate.value;

    if(taskToUpdate){
      const response = await axios.put(`http://localhost:9000/api/task/update-task/${taskToUpdate.id}`, {
        text: updatedTask.value,
      });
         // Mettez à jour la tâche dans la liste avec la nouvelle valeur
      const index = tasksList.value.findIndex(task => task.id === taskToUpdate.id);
      tasksList.value[index] = response.data;

      // Réinitialiser les valeurs
      taskForUpdate.value = null;
      updatedTask.value = '';
    } else {
      console.error('La tâche à mettre à jour est null');
    }
  } catch (error) {
    console.error('Erreur lors de la mise à jour de la tâche', error);
  }
};

const deleteTask = async (taskId: number) => {
  try {
    await axios.delete(`http://localhost:9000/api/task/delete-task/${taskId}`);
    // Filtrer la tâche supprimée de la liste
    tasksList.value = tasksList.value.filter(task => task.id !== taskId);
  } catch (error) {
    console.error('Erreur lors de la suppression de la tâche', error);
  }
};
</script>

<template>
    <main>
        <h3>Liste de vos tâches</h3>
          <!-- Formulaire de mise à jour -->
        <form v-if="taskForUpdate" @submit.prevent="updateTask">
            <label for="updatedTask">Mettre à jour une tâche :</label>
            <input v-model="updatedTask" id="updatedTask" type="text" required />
            <button type="submit" class="up">Mettre à jour</button>
        </form>

        <form @submit.prevent="addTask">
            <label for="newTask">Ajouter une tâche :</label>
            <input v-model="newTask" id="newTask" type="text" required />
            <button type="submit" class="add">Ajouter</button>
        </form>
        <div class="todoList">
          <ul v-if="tasksList.length > 0">
              <li v-for="task in tasksList" :key="task.id">
                  <p>{{ task.text }}</p>
                  <button @click="setTaskForUpdate(task)" class="update">Mettre à jour</button>
                  <button @click="deleteTask(task.id)" class="delete">Supprimer</button>
              </li>
          </ul>
          <p v-else class="listEmpty">Votre liste est vide</p>
        </div>
      </main>
  </template>
<style sass>
    @import "../assets/sass/style.sass";
</style>