<script setup>
import { ref, onMounted } from 'vue';
import { Preferences } from '@capacitor/preferences';
import { IonInput, IonButton, IonList, IonItem, IonCheckbox, IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';

const tasks = ref([]);  // Lista de tareas
const newTask = ref(''); // Nueva tarea

// Cargar tareas desde almacenamiento
const loadTasks = async () => {
  const storedTasks = await Preferences.get({ key: 'tasks' });
  tasks.value = storedTasks.value ? JSON.parse(storedTasks.value) : [];
};

// Guardar tareas en almacenamiento
const saveTasks = async () => {
  await Preferences.set({ key: 'tasks', value: JSON.stringify(tasks.value) });
};

// Agregar tarea
const addTask = () => {
  if (newTask.value.trim()) {
    tasks.value.push({ text: newTask.value, completed: false });
    tasks.value = [...tasks.value]; // Forzar reactividad
    newTask.value = ''; // Limpiar input
    saveTasks();
  }
};

// Eliminar tarea
const deleteTask = (index) => {
  tasks.value.splice(index, 1);
  tasks.value = [...tasks.value]; // Forzar reactividad
  saveTasks();
};

// Cambiar estado de tarea completada
const toggleTask = (task) => {
  task.completed = !task.completed;
  saveTasks();
};

const editingIndex = ref(null);

const startEditing = (index) => {
  editingIndex.value = index;
};

const stopEditing = () => {
  editingIndex.value = null;
  saveTasks();
};

onMounted(loadTasks);
</script>

<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Gestor de Tareas</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <div class="task-input-container">
        <ion-input v-model:model-value="newTask" placeholder="Inserta nueva tarea" class="task-input"></ion-input>
        <ion-button @click="addTask" expand="block" color="primary" class="add-button">Agregar</ion-button>
      </div>

      <ion-list>
        <ion-item v-for="(task, index) in tasks" :key="index" class="task-item">
          <ion-checkbox @ionChange="toggleTask(task)" :checked="task.completed" class="task-checkbox"></ion-checkbox>

          <template v-if="editingIndex === index">
            <ion-input v-model="task.text" @ionBlur="stopEditing" autofocus class="task-input-edit"></ion-input>
          </template>
          <template v-else>
            <span @click="startEditing(index)" :class="{ completed: task.completed }" class="task-text">
              {{ task.text }}
            </span>
          </template>

          <ion-button color="danger" @click="deleteTask(index)">Eliminar</ion-button>

        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<style scoped>
/* Estilos de tarea */
.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
  padding: 12px;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.task-checkbox {
  margin-right: 12px;
}

.task-text {
  flex-grow: 1;
  font-size: 16px;
  cursor: pointer;
  transition: color 0.3s;
}

.task-text.completed {
  text-decoration: line-through;
  color: gray;
}

.task-input-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.task-input {
  width: 80%;
  margin-right: 10px;
}

.add-button {
  width: 20%;
}

.task-input-edit {
  width: 100%;
  font-size: 16px;
}

ion-button {
  margin-left: 8px;
}

ion-button ion-icon {
  font-size: 20px;
}
</style>
