<script setup>
// page de connexion : envoi des identifiants au backend,
// récupération du token JWT puis stockage en localStorage
// si tout se passe bien, redirection vers le dashboard

import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import logoVideo from '../assets/logo.mp4'

const router = useRouter()

const email = ref('')
const password = ref('')
const errorMessage = ref('')
const loading = ref(false)

// connexion utilisateur + stockage du token
const handleLogin = async () => {
  errorMessage.value = ''
  loading.value = true

  try {
    const response = await axios.post('http://localhost:3000/auth/login', {
      email: email.value,
      password: password.value,
    })

    const token = response.data.access_token
    localStorage.setItem('token', token)

    router.push('/dashboard')
  } catch (error) {
    errorMessage.value = 'Email ou mot de passe incorrect.'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen flex bg-[#0B0B0C] text-[#F5F1E8]">
    <!-- Partie gauche -->
    <div
      class="hidden md:flex w-1/2 items-center justify-center bg-gradient-to-br from-[#18181B] via-[#0F0F11] to-[#09090A] p-12 border-r border-[#2A2A2E]"
    >
      <div class="max-w-md">
        <div class="mb-8">
          <video
            :src="logoVideo"
            autoplay
            muted
            loop
            playsinline
            class="w-72 h-auto object-contain opacity-95"
          ></video>
        </div>

        <h1 class="text-6xl font-bold mb-6 tracking-tight">
          Bienvenue sur <span class="text-[#E7DDD0]">Libheros !</span>
        </h1>

        <p class="text-[#B9B3A8] leading-relaxed text-lg">
          Une interface élégante pour gérer vos tâches, vos listes et votre
          organisation au quotidien.
        </p>
      </div>
    </div>

    <!-- Partie droite -->
    <div class="flex w-full md:w-1/2 items-center justify-center p-8">
      <div class="w-full max-w-md">
        <h2 class="text-5xl md:text-6xl font-bold mb-2 tracking-tight">
          Connexion
        </h2>

        <p class="text-[#B9B3A8] mb-8 text-base">
          Accédez à votre espace sécurisé
        </p>

        <form class="space-y-6" @submit.prevent="handleLogin">
          <div>
            <label class="block text-sm mb-2 text-[#E7DDD0]">Email</label>
            <input
              v-model="email"
              type="email"
              placeholder="votre@email.com"
              class="w-full px-4 py-3 rounded-2xl bg-[#141416] border border-[#2A2A2E] text-[#F5F1E8] placeholder:text-[#7D786F] focus:outline-none focus:border-[#E7DDD0] transition"
            />
          </div>

          <div>
            <label class="block text-sm mb-2 text-[#E7DDD0]">Mot de passe</label>
            <input
              v-model="password"
              type="password"
              placeholder="••••••••"
              class="w-full px-4 py-3 rounded-2xl bg-[#141416] border border-[#2A2A2E] text-[#F5F1E8] placeholder:text-[#7D786F] focus:outline-none focus:border-[#E7DDD0] transition"
            />
          </div>

          <p v-if="errorMessage" class="text-red-400 text-sm">
            {{ errorMessage }}
          </p>

          <button
            type="submit"
            :disabled="loading"
            class="w-full py-3 rounded-2xl bg-[#E7DDD0] text-[#111112] hover:bg-[#F5F1E8] transition font-semibold disabled:opacity-50"
          >
            {{ loading ? 'Connexion...' : 'Se connecter' }}
          </button>
        </form>

        <p class="text-sm text-[#B9B3A8] mt-6">
          Pas encore de compte ?
          <router-link to="/register" class="text-[#E7DDD0] hover:text-white ml-1">
            Créer un compte
          </router-link>
        </p>
      </div>
    </div>
  </div>
</template>