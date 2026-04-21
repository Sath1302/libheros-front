<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

const router = useRouter()

const email = ref('')
const password = ref('')
const errorMessage = ref('')
const loading = ref(false)

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
  <div class="min-h-screen flex bg-slate-950 text-white">
    <div class="hidden md:flex w-1/2 items-center justify-center bg-gradient-to-br from-indigo-900 via-slate-950 to-black p-12">
      <div class="max-w-md">
        <h1 class="text-4xl font-bold mb-6">
          Bienvenue sur <span class="text-indigo-400">Libheros</span>
        </h1>

        <p class="text-slate-400 leading-relaxed">
          La plateforme moderne pour gérer vos équipes, vos missions et vos aventures numériques.
        </p>
      </div>
    </div>

    <div class="flex w-full md:w-1/2 items-center justify-center">
      <div class="w-full max-w-md px-8">
        <h2 class="text-3xl font-bold mb-2">Connexion</h2>
        <p class="text-slate-400 mb-8">Accédez à votre espace sécurisé</p>

        <form class="space-y-6" @submit.prevent="handleLogin">
          <div>
            <label class="block text-sm mb-2">Email</label>
            <input
              v-model="email"
              type="email"
              placeholder="votre@email.com"
              class="w-full px-4 py-3 rounded-lg bg-slate-900 border border-slate-700 focus:outline-none focus:border-indigo-500"
            />
          </div>

          <div>
            <label class="block text-sm mb-2">Mot de passe</label>
            <input
              v-model="password"
              type="password"
              placeholder="••••••••"
              class="w-full px-4 py-3 rounded-lg bg-slate-900 border border-slate-700 focus:outline-none focus:border-indigo-500"
            />
          </div>

          <p v-if="errorMessage" class="text-red-400 text-sm">
            {{ errorMessage }}
          </p>

          <button
            type="submit"
            :disabled="loading"
            class="w-full py-3 rounded-lg bg-indigo-600 hover:bg-indigo-500 transition font-semibold disabled:opacity-50"
          >
            {{ loading ? 'Connexion...' : 'Se connecter' }}
          </button>
        </form>

        <p class="text-sm text-slate-400 mt-6">
          Pas encore de compte ?
          <router-link to="/register" class="text-indigo-400 hover:text-indigo-300 ml-1">
            Créer un compte
          </router-link>
        </p>
      </div>
    </div>
  </div>
</template>