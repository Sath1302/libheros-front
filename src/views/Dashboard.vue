<script setup>
// dashboard principal : je gère ici les listes, les tâches,
// les modals, la sidebar rétractable et le panneau de détail à droite
// l'idée était d'avoir toute la logique principale du projet sur cette page

import { computed, onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import logoVideo from '../assets/logo.mp4'

const router = useRouter()

const isSidebarCollapsed = ref(false)

const taskLists = ref([])
const selectedTaskListId = ref(null)
const tasks = ref([])
const loading = ref(true)
const errorMessage = ref('')

const showCreateTaskModal = ref(false)
const showEditTaskModal = ref(false)
const showCreateListModal = ref(false)
const showDeleteListModal = ref(false)
const showDeleteTaskModal = ref(false)
const showCompletedTasks = ref(false)
const showTaskDetailsSidebar = ref(false)

const editingTaskId = ref(null)
const deletingListId = ref(null)
const deletingListName = ref('')
const deletingTaskId = ref(null)
const deletingTaskName = ref('')
const selectedTask = ref(null)

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

const newList = ref({
  name: '',
})

const activeTasks = computed(() => tasks.value.filter((task) => !task.isCompleted))
const completedTasksList = computed(() => tasks.value.filter((task) => task.isCompleted))

const totalTasks = computed(() => tasks.value.length)
const completedTasks = computed(() => completedTasksList.value.length)
const pendingTasks = computed(() => activeTasks.value.length)

const selectedListName = computed(() => {
  return taskLists.value.find((list) => list.id === selectedTaskListId.value)?.name || 'Mes tâches'
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

// récupère les listes de l'utilisateur connecté
const fetchTaskLists = async () => {
  const response = await axios.get('http://localhost:3000/task-lists', {
    headers: getAuthHeaders(),
  })

  taskLists.value = response.data

  if (taskLists.value.length > 0) {
    const stillExists = taskLists.value.some((list) => list.id === selectedTaskListId.value)

    if (!selectedTaskListId.value || !stillExists) {
      selectedTaskListId.value = taskLists.value[0].id
    }
  } else {
    selectedTaskListId.value = null
  }
}

// récupère les tâches de la liste sélectionnée
const fetchTasksByList = async (taskListId) => {
  if (!taskListId) {
    tasks.value = []
    selectedTask.value = null
    showTaskDetailsSidebar.value = false
    return
  }

  const response = await axios.get(
    `http://localhost:3000/tasks/task-list/${taskListId}`,
    {
      headers: getAuthHeaders(),
    }
  )

  tasks.value = response.data

  if (selectedTask.value) {
    const updatedSelectedTask = tasks.value.find((task) => task.id === selectedTask.value.id)

    if (updatedSelectedTask) {
      selectedTask.value = updatedSelectedTask
    } else {
      selectedTask.value = null
      showTaskDetailsSidebar.value = false
    }
  }
}

const selectTaskList = async (taskListId) => {
  selectedTaskListId.value = taskListId
  loading.value = true
  errorMessage.value = ''
  selectedTask.value = null
  showTaskDetailsSidebar.value = false

  try {
    await fetchTasksByList(taskListId)
  } catch (error) {
    errorMessage.value = 'Impossible de récupérer les tâches.'
    console.error(error)
  } finally {
    loading.value = false
  }
}

const openTaskDetails = (task) => {
  selectedTask.value = task
  showTaskDetailsSidebar.value = true
}

const closeTaskDetails = () => {
  showTaskDetailsSidebar.value = false
  selectedTask.value = null
}

const createTask = async () => {
  errorMessage.value = ''

  if (!selectedTaskListId.value) {
    errorMessage.value = "Créez d'abord une liste avant d'ajouter une tâche."
    return
  }

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

const createList = async () => {
  errorMessage.value = ''

  if (!newList.value.name.trim()) {
    errorMessage.value = 'Le nom de la liste est requis.'
    return
  }

  try {
    await axios.post(
      'http://localhost:3000/task-lists',
      {
        name: newList.value.name,
      },
      {
        headers: getAuthHeaders(),
      }
    )

    showCreateListModal.value = false
    newList.value = { name: '' }

    await fetchTaskLists()

    if (selectedTaskListId.value) {
      await fetchTasksByList(selectedTaskListId.value)
    }
  } catch (error) {
    errorMessage.value = 'Impossible de créer la liste.'
    console.error(error)
  }
}

const openDeleteListModal = (list) => {
  deletingListId.value = list.id
  deletingListName.value = list.name
  showDeleteListModal.value = true
}

const deleteList = async () => {
  if (!deletingListId.value) return

  errorMessage.value = ''

  try {
    await axios.delete(`http://localhost:3000/task-lists/${deletingListId.value}`, {
      headers: getAuthHeaders(),
    })

    showDeleteListModal.value = false

    if (selectedTaskListId.value === deletingListId.value) {
      selectedTaskListId.value = null
      tasks.value = []
      selectedTask.value = null
      showTaskDetailsSidebar.value = false
    }

    deletingListId.value = null
    deletingListName.value = ''

    await fetchTaskLists()

    if (selectedTaskListId.value) {
      await fetchTasksByList(selectedTaskListId.value)
    } else {
      tasks.value = []
    }
  } catch (error) {
    errorMessage.value = 'Impossible de supprimer la liste.'
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

const openDeleteTaskModal = (task) => {
  deletingTaskId.value = task.id
  deletingTaskName.value = task.shortDescription
  showDeleteTaskModal.value = true
}

const deleteTask = async () => {
  if (!deletingTaskId.value) return

  try {
    await axios.delete(`http://localhost:3000/tasks/${deletingTaskId.value}`, {
      headers: getAuthHeaders(),
    })

    if (selectedTask.value?.id === deletingTaskId.value) {
      selectedTask.value = null
      showTaskDetailsSidebar.value = false
    }

    showDeleteTaskModal.value = false
    deletingTaskId.value = null
    deletingTaskName.value = ''

    await fetchTasksByList(selectedTaskListId.value)
  } catch (error) {
    errorMessage.value = 'Impossible de supprimer la tâche.'
    console.error(error)
  }
}

// ca lance le chargement principal du dashboard au montage
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

    <aside
      class="bg-[#121214]/85 backdrop-blur-xl border-r border-[#2A2A2E] p-6 flex flex-col shadow-2xl transition-all duration-300"
      :class="isSidebarCollapsed ? 'w-28' : 'w-80'"
    >
      <div>
        <div class="flex items-center justify-between mb-6">
          <video
            :src="logoVideo"
            autoplay
            muted
            loop
            playsinline
            class="h-auto opacity-95 object-contain transition-all duration-300"
            :class="isSidebarCollapsed ? 'w-12' : 'w-32'"
          ></video>

          <button
            @click="isSidebarCollapsed = !isSidebarCollapsed"
            class="h-10 w-10 rounded-xl bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] hover:bg-[#202024] transition shrink-0"
            :title="isSidebarCollapsed ? 'Déplier la sidebar' : 'Replier la sidebar'"
          >
            {{ isSidebarCollapsed ? '›' : '‹' }}
          </button>
        </div>

        <p
          v-if="!isSidebarCollapsed"
          class="text-[#B9B3A8] text-base mb-10"
        >
          Tableau de bord >
        </p>
      </div>

      <nav class="space-y-4">
        <div class="flex items-center justify-between mb-2">
          <div
            v-if="!isSidebarCollapsed"
            class="text-sm uppercase tracking-[0.2em] text-[#8E877D]"
          >
            Mes listes
          </div>

          <button
            @click="showCreateListModal = true"
            class="h-9 w-9 rounded-xl bg-[#E7DDD0] text-[#111112] font-bold hover:opacity-90 transition"
            title="Créer une liste"
          >
            +
          </button>
        </div>

        <div
          v-for="list in taskLists"
          :key="list.id"
          class="flex items-center gap-2"
        >
          <button
            @click="selectTaskList(list.id)"
            class="flex-1 text-left rounded-2xl border text-base transition"
            :class="[
              selectedTaskListId === list.id
                ? 'bg-[#E7DDD0] text-[#111112] font-semibold border-[#E7DDD0] shadow-lg'
                : 'bg-[#1A1A1D] border-[#2A2A2E] text-[#D4CEC3] hover:bg-[#202024] hover:text-white',
              isSidebarCollapsed ? 'px-3 py-4 text-center text-sm' : 'px-5 py-4'
            ]"
            :title="list.name"
          >
            <span v-if="!isSidebarCollapsed">{{ list.name }}</span>
            <span v-else>{{ list.name.charAt(0).toUpperCase() }}</span>
          </button>

          <button
            v-if="!isSidebarCollapsed"
            @click="openDeleteListModal(list)"
            class="h-12 w-12 rounded-2xl bg-[#2B1717] text-[#F5C2C2] hover:bg-[#3A1E1E] transition"
            title="Supprimer la liste"
          >
            🗑
          </button>
        </div>

        <div
          v-if="taskLists.length === 0 && !isSidebarCollapsed"
          class="rounded-2xl bg-[#1A1A1D] border border-dashed border-[#2A2A2E] p-4 text-sm text-[#8E877D]"
        >
          Aucune liste pour le moment. Créez-en une pour commencer.
        </div>

        <button
          class="w-full text-left rounded-2xl bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] transition hover:bg-[#202024] hover:text-white text-base"
          :class="isSidebarCollapsed ? 'px-3 py-4 text-center text-sm' : 'px-5 py-4'"
          title="Profil"
        >
          <span v-if="!isSidebarCollapsed">Profil</span>
          <span v-else>P</span>
        </button>
      </nav>

      <div class="mt-auto">
        <button
          @click="handleLogout"
          class="w-full rounded-2xl bg-[#2B1717] text-[#F5F1E8] hover:bg-[#3A1E1E] transition font-medium text-base border border-[#4A2626]"
          :class="isSidebarCollapsed ? 'px-3 py-4 text-sm' : 'py-4'"
          :title="isSidebarCollapsed ? 'Déconnexion' : ''"
        >
          <span v-if="!isSidebarCollapsed">Déconnexion</span>
          <span v-else>⎋</span>
        </button>
      </div>
    </aside>

    <main class="flex-1 p-10 transition-all duration-300" :class="showTaskDetailsSidebar ? 'mr-[420px]' : ''">
      <header class="flex flex-col md:flex-row md:items-start md:justify-between gap-6 mb-10">
        <div>
          <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] text-sm mb-5">
            <span class="h-2.5 w-2.5 rounded-full bg-[#E7DDD0]"></span>
            Espace connecté
          </div>

          <h2 class="text-5xl font-bold tracking-tight leading-tight">
            {{ selectedListName }}
          </h2>

          <p class="text-[#B9B3A8] mt-3 text-lg">
            Retrouvez les tâches de la liste sélectionnée.
          </p>
        </div>

        <div class="flex justify-end">
          <button
            @click="showCreateTaskModal = true"
            :disabled="!selectedTaskListId"
            class="px-6 py-3 rounded-xl bg-[#E7DDD0] text-black font-semibold hover:opacity-90 transition disabled:opacity-50 disabled:cursor-not-allowed"
          >
            + Nouvelle tâche
          </button>
        </div>
      </header>

      <section class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
        <div class="rounded-3xl bg-[#141416]/90 border border-[#2A2A2E] p-6 shadow-xl">
          <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-3">Total</p>
          <h3 class="text-4xl font-bold text-[#F5F1E8]">{{ totalTasks }}</h3>
          <p class="text-[#B9B3A8] mt-2">Nombre total de tâches</p>
        </div>

        <div class="rounded-3xl bg-[#141416]/90 border border-[#2A2A2E] p-6 shadow-xl">
          <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-3">Terminées</p>
          <h3 class="text-4xl font-bold text-[#B7E4C7]">{{ completedTasks }}</h3>
          <p class="text-[#B9B3A8] mt-2">Tâches déjà complétées</p>
        </div>

        <div class="rounded-3xl bg-[#141416]/90 border border-[#2A2A2E] p-6 shadow-xl">
          <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-3">En cours</p>
          <h3 class="text-4xl font-bold text-[#E7DDD0]">{{ pendingTasks }}</h3>
          <p class="text-[#B9B3A8] mt-2">Tâches encore à faire</p>
        </div>
      </section>

      <div v-if="loading" class="text-[#B9B3A8] text-lg">
        Chargement des tâches...
      </div>

      <div v-else-if="errorMessage" class="text-red-400 text-lg">
        {{ errorMessage }}
      </div>

      <div v-else-if="!selectedTaskListId" class="text-[#B9B3A8] text-lg">
        Créez ou sélectionnez une liste pour commencer.
      </div>

      <template v-else>
        <div v-if="activeTasks.length === 0" class="text-[#B9B3A8] text-lg mb-10">
          Aucune tâche en cours.
        </div>

        <section v-else class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-8 mb-10">
          <div
            v-for="task in activeTasks"
            :key="task.id"
            @click="openTaskDetails(task)"
            class="cursor-pointer rounded-3xl bg-[#141416]/90 backdrop-blur-xl border border-[#2A2A2E] p-8 shadow-xl transition hover:-translate-y-1 hover:border-[#3A3A40]"
          >
            <div class="mb-5 flex items-center justify-between gap-4">
              <h3 class="text-xl font-semibold flex-1 text-[#F5F1E8]">
                {{ task.shortDescription }}
              </h3>

              <div class="flex items-center gap-3">
                <button
                  @click.stop="toggleTaskStatus(task)"
                  class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg transition hover:scale-105 bg-[#2A241C] text-[#E7DDD0]"
                  title="Changer le statut"
                >
                  •
                </button>

                <button
                  @click.stop="openEditModal(task)"
                  class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#1E2330] text-[#D7E3FF] hover:bg-[#293246] transition hover:scale-105"
                  title="Modifier la tâche"
                >
                  ✏️
                </button>

                <button
                  @click.stop="openDeleteTaskModal(task)"
                  class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#2B1717] text-[#F5C2C2] hover:bg-[#3A1E1E] transition hover:scale-105"
                  title="Supprimer la tâche"
                >
                  🗑
                </button>
              </div>
            </div>

            <p class="text-base mb-4 text-[#B9B3A8]">
              {{ task.longDescription || 'Pas de description.' }}
            </p>

            <div class="text-sm text-[#8E877D] space-y-2">
              <p>
                Statut :
                <span class="text-[#DCCFC1]">En cours</span>
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

        <section class="mt-6">
          <button
            @click="showCompletedTasks = !showCompletedTasks"
            class="mb-6 inline-flex items-center gap-3 px-5 py-3 rounded-2xl bg-[#141416]/90 border border-[#2A2A2E] text-[#F5F1E8] hover:border-[#3A3A40] transition"
          >
            <span class="text-lg">{{ showCompletedTasks ? '▾' : '▸' }}</span>
            <span class="font-semibold">
              Mes tâches terminées ({{ completedTasksList.length }})
            </span>
          </button>

          <div v-if="showCompletedTasks">
            <div v-if="completedTasksList.length === 0" class="text-[#8E877D] text-lg">
              Aucune tâche terminée.
            </div>

            <div v-else class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-8">
              <div
                v-for="task in completedTasksList"
                :key="task.id"
                @click="openTaskDetails(task)"
                class="cursor-pointer rounded-3xl bg-[#141416]/70 backdrop-blur-xl border border-[#263128] p-8 shadow-xl opacity-95"
              >
                <div class="mb-5 flex items-center justify-between gap-4">
                  <h3 class="text-xl font-semibold flex-1 text-[#8E877D] line-through">
                    {{ task.shortDescription }}
                  </h3>

                  <div class="flex items-center gap-3">
                    <button
                      @click.stop="toggleTaskStatus(task)"
                      class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg transition hover:scale-105 bg-[#1F3328] text-[#B7E4C7]"
                      title="Remettre en cours"
                    >
                      ✓
                    </button>

                    <button
                      @click.stop="openEditModal(task)"
                      class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#1E2330] text-[#D7E3FF] hover:bg-[#293246] transition hover:scale-105"
                      title="Modifier la tâche"
                    >
                      ✏️
                    </button>

                    <button
                      @click.stop="openDeleteTaskModal(task)"
                      class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg bg-[#2B1717] text-[#F5C2C2] hover:bg-[#3A1E1E] transition hover:scale-105"
                      title="Supprimer la tâche"
                    >
                      🗑
                    </button>
                  </div>
                </div>

                <p class="text-base mb-4 text-[#7B756C]">
                  {{ task.longDescription || 'Pas de description.' }}
                </p>

                <div class="text-sm text-[#8E877D] space-y-2">
                  <p>
                    Statut :
                    <span class="text-[#B7E4C7]">Terminée</span>
                  </p>

                  <p v-if="task.dueDate">
                    Échéance :
                    <span class="text-[#DCCFC1]">
                      {{ new Date(task.dueDate).toLocaleDateString() }}
                    </span>
                  </p>
                </div>
              </div>
            </div>
          </div>
        </section>
      </template>
    </main>

    <aside
      v-if="showTaskDetailsSidebar && selectedTask"
      class="fixed top-0 right-0 h-full w-[420px] bg-[#111112]/95 backdrop-blur-2xl border-l border-[#2A2A2E] shadow-2xl z-40 p-8 overflow-y-auto"
    >
      <div class="flex items-start justify-between gap-4 mb-8">
        <div>
          <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-3">
            Détail de la tâche
          </p>
          <h2
            class="text-3xl font-bold leading-tight"
            :class="selectedTask.isCompleted ? 'text-[#8E877D] line-through' : 'text-[#F5F1E8]'"
          >
            {{ selectedTask.shortDescription }}
          </h2>
        </div>

        <button
          @click="closeTaskDetails"
          class="h-11 w-11 rounded-2xl bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] hover:bg-[#202024] transition"
          title="Fermer"
        >
          ✕
        </button>
      </div>

      <div class="space-y-6">
        <div class="rounded-3xl bg-[#141416]/90 border border-[#2A2A2E] p-6">
          <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-3">Description</p>
          <p class="text-[#D4CEC3] leading-7">
            {{ selectedTask.longDescription || 'Pas de description.' }}
          </p>
        </div>

        <div class="rounded-3xl bg-[#141416]/90 border border-[#2A2A2E] p-6 space-y-4">
          <div>
            <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-2">Statut</p>
            <p :class="selectedTask.isCompleted ? 'text-[#B7E4C7]' : 'text-[#E7DDD0]'">
              {{ selectedTask.isCompleted ? 'Terminée' : 'En cours' }}
            </p>
          </div>

          <div v-if="selectedTask.dueDate">
            <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-2">Date d’échéance</p>
            <p class="text-[#D4CEC3]">
              {{ new Date(selectedTask.dueDate).toLocaleDateString() }}
            </p>
          </div>

          <div v-if="selectedTask.createdAt">
            <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-2">Date de création</p>
            <p class="text-[#D4CEC3]">
              {{ new Date(selectedTask.createdAt).toLocaleDateString() }}
            </p>
          </div>

          <div v-if="selectedTask.taskList?.name">
            <p class="text-sm uppercase tracking-[0.18em] text-[#8E877D] mb-2">Liste</p>
            <p class="text-[#D4CEC3]">
              {{ selectedTask.taskList.name }}
            </p>
          </div>
        </div>

        <div class="grid grid-cols-2 gap-4">
          <button
            @click="toggleTaskStatus(selectedTask)"
            class="py-3 rounded-2xl font-semibold transition"
            :class="
              selectedTask.isCompleted
                ? 'bg-[#1F3328] text-[#B7E4C7] hover:opacity-90'
                : 'bg-[#2A241C] text-[#E7DDD0] hover:opacity-90'
            "
          >
            {{ selectedTask.isCompleted ? 'Remettre en cours' : 'Marquer terminée' }}
          </button>

          <button
            @click="openEditModal(selectedTask)"
            class="py-3 rounded-2xl bg-[#1E2330] text-[#D7E3FF] font-semibold hover:opacity-90 transition"
          >
            Modifier
          </button>
        </div>

        <button
          @click="openDeleteTaskModal(selectedTask)"
          class="w-full py-3 rounded-2xl bg-[#8B1E1E] text-white font-semibold hover:bg-[#A32424] transition"
        >
          Supprimer la tâche
        </button>
      </div>
    </aside>

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

    <div
      v-if="showCreateListModal"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
    >
      <div class="bg-[#111112] border border-[#2A2A2E] p-8 rounded-2xl w-[420px] space-y-4 shadow-2xl">
        <h2 class="text-2xl font-bold">Créer une liste</h2>

        <input
          v-model="newList.name"
          placeholder="Nom de la liste"
          class="w-full p-3 rounded-lg bg-[#1A1A1D] border border-[#2A2A2E] text-[#F5F1E8] outline-none"
        />

        <div class="flex justify-end gap-3">
          <button
            @click="showCreateListModal = false"
            class="px-4 py-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
          >
            Annuler
          </button>

          <button
            @click="createList"
            class="px-4 py-2 rounded-lg bg-[#E7DDD0] text-black font-semibold hover:opacity-90 transition"
          >
            Créer
          </button>
        </div>
      </div>
    </div>

    <div
      v-if="showDeleteListModal"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
    >
      <div class="bg-[#111112] border border-[#2A2A2E] p-8 rounded-2xl w-[460px] space-y-4 shadow-2xl">
        <h2 class="text-2xl font-bold text-[#F5F1E8]">Supprimer la liste</h2>

        <p class="text-[#B9B3A8] leading-relaxed">
          Voulez-vous vraiment supprimer la liste
          <span class="text-[#F5F1E8] font-semibold">"{{ deletingListName }}"</span> ?
        </p>

        <p class="text-red-400 text-sm">
          Toutes les tâches associées à cette liste seront également supprimées.
        </p>

        <div class="flex justify-end gap-3 pt-2">
          <button
            @click="showDeleteListModal = false"
            class="px-4 py-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
          >
            Annuler
          </button>

          <button
            @click="deleteList"
            class="px-4 py-2 rounded-lg bg-[#8B1E1E] text-white font-semibold hover:bg-[#A32424] transition"
          >
            Supprimer
          </button>
        </div>
      </div>
    </div>

    <div
      v-if="showDeleteTaskModal"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
    >
      <div class="bg-[#111112] border border-[#2A2A2E] p-8 rounded-2xl w-[460px] space-y-4 shadow-2xl">
        <h2 class="text-2xl font-bold text-[#F5F1E8]">Supprimer la tâche</h2>

        <p class="text-[#B9B3A8] leading-relaxed">
          Voulez-vous vraiment supprimer la tâche
          <span class="text-[#F5F1E8] font-semibold">"{{ deletingTaskName }}"</span> ?
        </p>

        <p class="text-red-400 text-sm">
          Cette action est définitive.
        </p>

        <div class="flex justify-end gap-3 pt-2">
          <button
            @click="showDeleteTaskModal = false"
            class="px-4 py-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
          >
            Annuler
          </button>

          <button
            @click="deleteTask"
            class="px-4 py-2 rounded-lg bg-[#8B1E1E] text-white font-semibold hover:bg-[#A32424] transition"
          >
            Supprimer
          </button>
        </div>
      </div>
    </div>
  </div>
</template>