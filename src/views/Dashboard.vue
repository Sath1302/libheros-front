<script setup>
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import logoVideo from '../assets/logo.mp4'

const router = useRouter()

const tasks = ref([])
const loading = ref(true)
const errorMessage = ref('')

const handleLogout = () => {
  localStorage.removeItem('token')
  router.push('/login')
}

const fetchTasks = async () => {
  loading.value = true
  errorMessage.value = ''

  try {
    const token = localStorage.getItem('token')

    const taskListsResponse = await axios.get('http://localhost:3000/task-lists', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    })

    const taskLists = taskListsResponse.data

    if (!taskLists.length) {
      tasks.value = []
      return
    }

    const firstTaskListId = taskLists[0].id

    const tasksResponse = await axios.get(
      `http://localhost:3000/tasks/task-list/${firstTaskListId}`,
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    )

    tasks.value = tasksResponse.data
  } catch (error) {
    errorMessage.value = 'Impossible de récupérer les tâches.'
    console.error(error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchTasks()
})
</script>

<template>
  <div class="min-h-screen bg-[#0B0B0C] text-[#F5F1E8] flex relative overflow-hidden">

    <!-- Fond -->
    <div class="absolute inset-0 -z-10">
      <div class="absolute top-0 left-0 h-96 w-96 rounded-full bg-white/5 blur-3xl"></div>
      <div class="absolute bottom-0 right-0 h-96 w-96 rounded-full bg-[#E7DDD0]/5 blur-3xl"></div>
    </div>

    <!-- Sidebar -->
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
        <button class="w-full text-left px-5 py-4 rounded-2xl bg-[#E7DDD0] text-[#111112] font-semibold transition hover:scale-[1.02] text-base shadow-lg">
          Mes tâches
        </button>

        <button class="w-full text-left px-5 py-4 rounded-2xl bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] transition hover:bg-[#202024] hover:text-white text-base">
          Mes listes
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

    <!-- Contenu principal -->
    <main class="flex-1 p-10">
      <header class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-10">
        <div>
          <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-[#1A1A1D] border border-[#2A2A2E] text-[#D4CEC3] text-sm mb-5">
            <span class="h-2.5 w-2.5 rounded-full bg-[#E7DDD0]"></span>
            Espace connecté
          </div>

          <h2 class="text-5xl font-bold tracking-tight leading-tight">
            Mes tâches
          </h2>

          <p class="text-[#B9B3A8] mt-3 text-lg">
            Retrouvez toutes vos tâches enregistrées.
          </p>
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
          <div class="mb-5 flex items-center justify-between">
            <h3 class="text-xl font-semibold text-[#F5F1E8]">
              {{ task.shortDescription }}
            </h3>

            <div
              class="h-12 w-12 rounded-2xl flex items-center justify-center text-lg"
              :class="task.isCompleted ? 'bg-[#1F3328] text-[#B7E4C7]' : 'bg-[#2A241C] text-[#E7DDD0]'"
            >
              {{ task.isCompleted ? '✓' : '•' }}
            </div>
          </div>

          <p class="text-[#B9B3A8] text-base mb-4">
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
  </div>
</template>