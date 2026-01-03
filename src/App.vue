<script setup lang="ts">
import { ref } from 'vue'
import draggable from 'vuedraggable'

// --- TYPER ---
interface Task {
  id: number;
  title: string;
  description: string;
}

interface Column {
  id: number;
  title: string;
  tasks: Task[];
}

// --- DATA ---
const columns = ref<Column[]>([
  { id: 1, title: 'Recruiting (To Do)', tasks: [{ id: 101, title: 'Name of your mission', description: 'Describe your mission!' }] },
  { id: 2, title: 'Commando Ops (Progress)', tasks: [] },
  { id: 3, title: 'Mission Accomplished (Done)', tasks: [] }
])

const dialog = ref(false)
const isEditing = ref(false)
// Her sikrer vi, at titleText findes, så v-model i din dialog virker!
const taskForm = ref({ id: 0, titleText: '', description: '' })
const activeColumn = ref<Column | null>(null)

// --- LOGIK ---
const openAddDialog = (column: Column) => {
  isEditing.value = false
  activeColumn.value = column
  taskForm.value = { id: 0, titleText: '', description: '' }
  dialog.value = true
}

const openEditDialog = (column: Column, task: Task) => {
  isEditing.value = true
  activeColumn.value = column
  taskForm.value = { id: task.id, titleText: task.title, description: task.description }
  dialog.value = true
}

const saveTask = () => {
  // Tjekker om der er valgt en kolonne og om titlen er udfyldt
  if (!activeColumn.value || !taskForm.value.titleText) return

  if (isEditing.value) {
    // REDIGER: Find opgaven via ID
    const index = activeColumn.value.tasks.findIndex(t => t.id === taskForm.value.id)
    if (index !== -1) {
      activeColumn.value.tasks[index] = { 
        id: taskForm.value.id, 
        title: taskForm.value.titleText, 
        description: taskForm.value.description 
      }
    }
  } else {
    // TILFØJ: Opret ny opgave med unikt ID
    activeColumn.value.tasks.push({
      id: Date.now(),
      title: taskForm.value.titleText,
      description: taskForm.value.description
    })
  }
  
  // Luk vinduet efter gem
  dialog.value = false
}

const deleteTask = (column: Column, taskId: number) => {
  column.tasks = column.tasks.filter(t => t.id !== taskId)
}
</script>

<template>
  <v-app>
    <v-main style="background: linear-gradient(135deg, #1a1a1a 0%, #3d2b1f 100%); min-height: 100vh;">
      
      <v-container class="d-flex flex-column align-center pt-10" fluid>
        
        <div class="text-center mb-8">
          <h1 class="tiger-title text-h2 font-weight-black italic">
            TIGER <span class="text-orange-darken-2">MAFIA</span>
          </h1>
          <p class="text-white text-subtitle-1 font-italic font-weight-bold">
            "ACTION IS COMING!" 🎬
          </p>
        </div>

        <div class="d-flex flex-row align-start justify-center overflow-x-auto pb-10 w-100 px-4">
          <v-card v-for="column in columns" :key="column.id" width="340" class="mx-4 rounded-xl elevation-10 column-box">
            <v-toolbar color="black" density="comfortable" flat>
              <v-toolbar-title class="text-orange-darken-2 font-weight-black text-uppercase tracking-widest">
                {{ column.title }}
              </v-toolbar-title>
            </v-toolbar>

            <v-card-text class="pa-4 bg-grey-darken-4" style="min-height: 500px;">
              <draggable v-model="column.tasks" group="tasks" item-key="id" class="drag-zone">
                <template #item="{ element }">
                  <v-card class="mb-4 tiger-task-card" elevation="4" @click="openEditDialog(column, element)">
    <v-card-item>
    <v-card-title class="font-weight-black text-uppercase" style="font-size: 0.9rem;">
      🐅 {{ element.title }}
    </v-card-title>
    <v-card-subtitle class="text-white text-wrap opacity-80 mt-1">
      {{ element.description }}
    </v-card-subtitle>
  </v-card-item>
  <v-card-actions>
    <v-icon size="small" color="grey-darken-1" class="ml-2">mdi-pencil</v-icon>
    <v-spacer></v-spacer>
    <v-btn icon="mdi-sword-cross" size="small" color="orange" title="Delete Task" @click.stop="deleteTask(column, element.id)"></v-btn>
  </v-card-actions>
</v-card>
                </template>
              </draggable>

              <v-btn block color="orange-darken-4" variant="elevated" class="mt-4 rounded-lg font-weight-black tiger-button" size="large" @click="openAddDialog(column)">
                New task!
              </v-btn>
            </v-card-text>
          </v-card>
        </div>
      </v-container>
    </v-main>

    <v-dialog v-model="dialog" max-width="450">
  <v-card class="bg-grey-darken-4 text-white rounded-xl border-orange">
    <v-card-title class="bg-orange-darken-4 font-weight-black">
      {{ isEditing ? 'UPDATE INTEL' : 'COMMANDO LOGBOOK' }}
    </v-card-title>
    <v-card-text class="pt-6">
      <v-text-field v-model="taskForm.titleText" label="Mission Title" variant="solo-filled" bg-color="grey-darken-3" color="orange"></v-text-field>
      <v-textarea v-model="taskForm.description" label="Mission Intel" variant="solo-filled" bg-color="grey-darken-3" color="orange"></v-textarea>
    </v-card-text>
    <v-card-actions class="pa-6">
      <v-btn variant="text" color="white" @click="dialog = false">RETREAT</v-btn>
      <v-spacer></v-spacer>
      <v-btn color="orange-darken-2" variant="elevated" class="px-8 font-weight-bold" @click="saveTask">
        {{ isEditing ? 'UPDATE' : 'DEPLOY' }}
      </v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
  </v-app>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Permanent+Marker&display=swap');

.tiger-title {
  font-family: 'Permanent Marker', cursive !important;
  color: white;
  letter-spacing: 5px;
  text-shadow: 4px 4px 0px #FF8C00;
}

.column-box {
  border: 2px solid #333 !important;
  background-color: #121212 !important;
}

/* Kort med Gradient */
.tiger-task-card {
  background: linear-gradient(135deg, #2b2b2b 0%, #1a1a1a 100%) !important;
  border-left: 6px solid #FF8C00 !important;
  color: white !important;
  transition: all 0.3s ease;
}

.tiger-task-card:hover {
  cursor: grab;
  transform: rotate(-1deg) scale(1.05);
  box-shadow: 0 0 20px rgba(255, 140, 0, 0.4) !important;
}

.tiger-button {
  text-shadow: 1px 1px 0px black;
  border: 2px solid #FF8C00;
}

.drag-zone {
  min-height: 400px;
}

.border-orange {
  border: 3px solid #FF8C00 !important;
}
</style>