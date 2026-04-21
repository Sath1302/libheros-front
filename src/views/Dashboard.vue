<script setup>
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import logoVideo from '../assets/logo.mp4'

const router = useRouter()

const taskLists = ref([])
const selectedTaskListId = ref(null)
const tasks = ref([])
const loading = ref(true)
const errorMessage = ref('')

const showCreateTaskModal = ref(false)
const showEditTaskModal = ref(false)
const editingTaskId = ref(null)

const newTask = ref({
  shortDescription: '',
  longDescription: '',
  dueDate: '',
})

const editTask = ref({
  shortDescription: '',
  longDescription: '',
  dueDate: '',
})

const handleLogout = () => {
  localStorage.removeItem('token')
  router.push('/login')
}

const getAuthHeaders = () => {
  const token = localStorage.getItem('token')

  return {
    Authorization: `Bearer ${token}`,
  }
}

const fetchTaskLists = async () => {
  const response = await axios.get('http://localhost:3000/task-lists', {
    headers: getAuthHeaders(),
  })

  taskLists.value = response.data

  if (taskLists.value.length > 0 && !selectedTaskListId.value) {
    selectedTaskListId.value = taskLists.value[0].id
  }
}

const fetchTasksByList = async (taskListId) => {
  const response = await axios.get(
    `http://localhost:3000/tasks/task-list/${taskListId}`,
    {
      headers: getAuthHeaders(),
    }
  )

  tasks.value = response.data
}

const selectTaskList = async (taskListId) => {
  selectedTaskListId.value = taskListId
  loading.value = true
  errorMessage.value = ''

  try {
    await fetchTasksByList(taskListId)
  } catch (error) {
    errorMessage.value = 'Impossible de récupérer les tâches.'
    console.error(error)
  } finally {
    loading.value = false
  }
}

const createTask = async () => {
  errorMessage.value = ''

  try {
    await axios.post(
      'http://localhost:3000/tasks',
      {
        shortDescription: newTask.value.shortDescription,
        longDescription: newTask.value.longDescription,
        dueDate: newTask.value.dueDate,
        taskListId: selectedTaskListId.value,
      },
      {
        headers: getAuthHeaders(),
      }
    )

    showCreateTaskModal.value = false

    newTask.value = {
      shortDescription: '',
      longDescription: '',
      dueDate: '',
    }

    await fetchTasksByList(selectedTaskListId.value)
  } catch (error) {
    errorMessage.value = 'Impossible de créer la tâche.'
    console.error(error)
  }
}

const openEditModal = (task) => {
  editingTaskId.value = task.id
  editTask.value = {
    shortDescription: task.shortDescription || '',
    longDescription: task.longDescription || '',
    dueDate: task.dueDate ? new Date(task.dueDate).toISOString().split('T')[0] : '',
  }
  showEditTaskModal.value = true
}

const updateTask = async () => {
  errorMessage.value = ''

  try {
    await axios.patch(
      `http://localhost:3000/tasks/${editingTaskId.value}`,
      {
        shortDescription: editTask.value.shortDescription,
        longDescription: editTask.value.longDescription,
        dueDate: editTask.value.dueDate,
      },
      {
        headers: getAuthHeaders(),
      }
    )

    showEditTaskModal.value = false
    editingTaskId.value = null

    editTask.value = {
      shortDescription: '',
      longDescription: '',
      dueDate: '',
    }

    await fetchTasksByList(selectedTaskListId.value)
  } catch (error) {
    errorMessage.value = 'Impossible de modifier la tâche.'
    console.error(error)
  }
}

const toggleTaskStatus = async (task) => {
  try {
    await axios.patch(
      `http://localhost:3000/tasks/${task.id}`,
      {
        isCompleted: !task.isCompleted,
      },
      {
        headers: getAuthHeaders(),
      }
    )

    await fetchTasksByList(selectedTaskListId.value)
  } catch (error) {
    errorMessage.value = 'Impossible de mettre à jour la tâche.'
    console.error(error)
  }
}

const deleteTask = async (taskId) => {
  const confirmed = window.confirm('Voulez-vous vraiment supprimer cette tâche ?')

  if (!confirmed) return

  try {
    await axios.delete(`http://localhost:3000/tasks/${taskId}`, {
      headers: getAuthHeaders(),
    })

    await fetchTasksByList(selectedTaskListId.value)
  } catch (error) {
    errorMessage.value = 'Impossible de supprimer la tâche.'
    console.error(error)
  }
}

const initDashboard = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    await fetchTaskLists()

    if (selectedTaskListId.value) {
      await fetchTasksByList(selectedTaskListId.value)
    } else {
      tasks.value = []
    }
  } catch (error) {
    errorMessage.value = 'Impossible de charger le tableau de bord.'
    console.error(error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  initDashboard()
})
</script>

<template>
  <div class="min-h-screen bg-[#0B0B0C] text-[#F5F1E8] flex relative overflow-hidden">
    <div class="absolute inset-0 -z-10">
      <div class="absolute top-0 left-0 h-96 w-96 rounded-full bg-white/5 blur-3xl"></div>
      <div class="absolute bottom-0 right-0 h-96 w-96 rounded-full bg-[#E7DDD0]/5 blur-3xl"></div>
    </div>

    <aside class="w-80 bg-[#121214]/85 backdrop-blur-xl border-r border-[#2A2A2E] p-8 flex flex-col shadow-2xl">
      <div>
        <video
          :src="logoVideo"
          autoplay
          muted
          loop
          playsinline
          class="w-32 h-auto mb-6 opacity-95 object-contain"
        ></video>

        <p class="text-[#B9B3A8] text-base mb-10">
          Tableau de bord
        </p>
      </div>

      <nav class="space-y-4">
        <div class="text-sm uppercase tracking-[0.2em] text-[#8E877D] mb-2">
          Mes listes
        </div>

        <button
          v-for="list in taskLists"
          :key="list.id"
          @click="selectTaskList(list.id)"
          class="w-full text-left px-5 py-4 rounded-2xl border text-base transition"
          :class="
            selectedTaskListId === list.id
              ? 'bg-[#E7DDD0] text-[#111112] font-semibold border-[#E7DDD0] shadow-lg'
              : 'bg-[#1A1A1D] border-[#2A2A2E] text-[#D4CEC3] hover:bg-[#202024] hover:text-white'
          "
        >
          {{ list.name }}
        </button>

        <button class="w-full text-left px-5 py-4 rounded-2xl bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] transition hover:bg-[#202024] hover:text-white text-base">
          Profil
        </button>
      </nav>

      <div class="mt-auto">
        <button
          @click="handleLogout"
          class="w-full py-4 rounded-2xl bg-[#2B1717] text-[#F5F1E8] hover:bg-[#3A1E1E] transition font-medium text-base border border-[#4A2626]"
        >
          Déconnexion
        </button>
      </div>
    </aside>

    <main class="flex-1 p-10">
      <header class="flex flex-col md:flex-row md:items-start md:justify-between gap-6 mb-10">
        <div>
          <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] text-sm mb-5">
            <span class="h-2.5 w-2.5 rounded-full bg-[#E7DDD0]"></span>
            Espace connecté
          </div>

          <h2 class="text-5xl font-bold tracking-tight leading-tight">
            {{ taskLists.find((list) => list.id === selectedTaskListId)?.name || 'Mes tâches' }}
          </h2>

          <p class="text-[#B9B3A8] mt-3 text-lg">
            Retrouvez les tâches de la liste sélectionnée.
          </p>
        </div>

        <div class="flex justify-end">
          <button
            @click="showCreateTaskModal = true"
            class="px-6 py-3 rounded-xl bg-[#E7DDD0] text-black font-semibold hover:opacity-90 transition"
          >
            + Nouvelle tâche
          </button>
        </div>
      </header>

      <div v-if="loading" class="text-[#B9B3A8] text-lg">
        Chargement des tâches...
      </div>

      <div v-else-if="errorMessage" class="text-red-400 text-lg">
        {{ errorMessage }}
      </div>

      <div v-else-if="tasks.length === 0" class="text-[#B9B3A8] text-lg">
        Aucune tâche trouvée.
      </div>

      <section v-else class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-8">
        <div
          v-for="task in tasks"
          :key="task.id"
          class="rounded-3xl bg-[#141416]/90 backdrop-blur-xl border border-[#2A2A2E] p-8 shadow-xl transition hover:-translate-y-1 hover:border-[#3A3A40]"
        >
          <div class="mb-5 flex items-center justify-between gap-4">
            <h3
              class="text-xl font-semibold flex-1"
              :class="task.isCompleted ? 'text-[#8E877D] line-through' : 'text-[#F5F1E8]'"
            >
              {{ task.shortDescription }}
            </h3>

            <div class="flex items-center gap-3">
              <button
                @click="toggleTaskStatus(task)"
                class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg transition hover:scale-105"
                :class="task.isCompleted ? 'bg-[#1F3328] text-[#B7E4C7]' : 'bg-[#2A241C] text-[#E7DDD0]'"
                title="Changer le statut"
              >
                {{ task.isCompleted ? '✓' : '•' }}
              </button>

              <button
                @click="openEditModal(task)"
                class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#1E2330] text-[#D7E3FF] hover:bg-[#293246] transition hover:scale-105"
                title="Modifier la tâche"
              >
                ✏️
              </button>

              <button
                @click="deleteTask(task.id)"
                class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#2B1717] text-[#F5C2C2] hover:bg-[#3A1E1E] transition hover:scale-105"
                title="Supprimer la tâche"
              >
                🗑
              </button>
            </div>
          </div>

          <p
            class="text-base mb-4"
            :class="task.isCompleted ? 'text-[#7B756C]' : 'text-[#B9B3A8]'"
          >
            {{ task.longDescription || 'Pas de description.' }}
          </p>

          <div class="text-sm text-[#8E877D] space-y-2">
            <p>
              Statut :
              <span class="text-[#DCCFC1]">
                {{ task.isCompleted ? 'Terminée' : 'En cours' }}
              </span>
            </p>

            <p v-if="task.dueDate">
              Échéance :
              <span class="text-[#DCCFC1]">
                {{ new Date(task.dueDate).toLocaleDateString() }}
              </span>
            </p>
          </div>
        </div>
      </section>
    </main>

    <div
      v-if="showCreateTaskModal"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
    >
      <div class="bg-[#111112] border border-[#2A2A2E] p-8 rounded-2xl w-[500px] space-y-4 shadow-2xl">
        <h2 class="text-2xl font-bold">Créer une tâche</h2>

        <input
          v-model="newTask.shortDescription"
          placeholder="Titre"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none"
        />

        <textarea
          v-model="newTask.longDescription"
          placeholder="Description"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none min-h-[120px]"
        ></textarea>

        <input
          type="date"
          v-model="newTask.dueDate"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none"
        />

        <div class="flex justify-end gap-3">
          <button
            @click="showCreateTaskModal = false"
            class="px-4 py-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
          >
            Annuler
          </button>

          <button
            @click="createTask"
            class="px-4 py-2 rounded-lg bg-[#E7DDD0] text-black font-semibold hover:opacity-90 transition"
          >
            Créer
          </button>
        </div>
      </div>
    </div>

    <div
      v-if="showEditTaskModal"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
    >
      <div class="bg-[#111112] border border-[#2A2A2E] p-8 rounded-2xl w-[500px] space-y-4 shadow-2xl">
        <h2 class="text-2xl font-bold">Modifier la tâche</h2>

        <input
          v-model="editTask.shortDescription"
          placeholder="Titre"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none"
        />

        <textarea
          v-model="editTask.longDescription"
          placeholder="Description"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none min-h-[120px]"
        ></textarea>

        <input
          type="date"
          v-model="editTask.dueDate"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none"
        />

        <div class="flex justify-end gap-3">
          <button
            @click="showEditTaskModal = false"
            class="px-4 py-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
          >
            Annuler
          </button>

          <button
            @click="updateTask"
            class="px-4 py-2 rounded-lg bg-[#E7DDD0] text-black font-semibold hover:opacity-90 transition"
          >
            Enregistrer
          </button>
        </div>
      </div>
    </div>
  </div>
</template>