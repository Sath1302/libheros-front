<script setup>
// page d'inscription : je vérifie d'abord les champs côté front
// puis j'envoie les infos au backend pour créer le compte
// si ça marche, je redirige ensuite vers la page de connexion

import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import logoVideo from '../assets/logo.mp4'

const router = useRouter()

const form = ref({
  firstName: '',
  lastName: '',
  email: '',
  confirmEmail: '',
  password: '',
  confirmPassword: '',
})

const loading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')

// petites vérifications côté front avant d'envoyer l'inscription
const handleRegister = async () => {
  errorMessage.value = ''
  successMessage.value = ''

  if (
    !form.value.firstName ||
    !form.value.lastName ||
    !form.value.email ||
    !form.value.confirmEmail ||
    !form.value.password ||
    !form.value.confirmPassword
  ) {
    errorMessage.value = 'Merci de remplir tous les champs.'
    return
  }

  if (form.value.email !== form.value.confirmEmail) {
    errorMessage.value = 'Les adresses email ne correspondent pas.'
    return
  }

  if (form.value.password !== form.value.confirmPassword) {
    errorMessage.value = 'Les mots de passe ne correspondent pas.'
    return
  }

  try {
    loading.value = true

    await axios.post('http://localhost:3000/auth/register', {
      firstName: form.value.firstName,
      lastName: form.value.lastName,
      email: form.value.email,
      password: form.value.password,
    })

    successMessage.value = 'Compte créé avec succès. Redirection vers la connexion...'

    setTimeout(() => {
      router.push('/login')
    }, 1200)
  } catch (error) {
    console.error(error)
    errorMessage.value =
      error?.response?.data?.message || "Impossible de créer le compte."
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-[#0B0B0C] text-[#F5F1E8] flex">
    <div class="hidden md:flex w-1/2 items-center justify-center bg-gradient-to-br from-[#111216] via-[#0B0B0C] to-black p-12 border-r border-[#1F1F22]">
      <div class="max-w-md">
        <video
          :src="logoVideo"
          autoplay
          muted
          loop
          playsinline
          class="w-48 h-auto mb-8 object-contain"
        ></video>

        <h1 class="text-5xl font-bold leading-tight mb-6">
          Rejoignez <span class="text-[#E7DDD0]">Libheros</span>
        </h1>

        <p class="text-[#B9B3A8] text-lg leading-8">
          Créez votre espace personnel pour organiser vos tâches, vos listes et
          votre quotidien avec une interface moderne et élégante.
        </p>
      </div>
    </div>

    <div class="flex w-full md:w-1/2 items-center justify-center p-8 md:p-16">
      <div class="w-full max-w-xl">
        <div class="mb-8">
          <h2 class="text-4xl font-bold mb-2">Créer un compte</h2>
          <p class="text-[#B9B3A8] text-lg">Remplissez les informations ci-dessous.</p>
        </div>

        <div class="space-y-5">
          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label class="block mb-2 text-sm text-[#D8D2C8]">Prénom</label>
              <input
                v-model="form.firstName"
                type="text"
                placeholder="Votre prénom"
                class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
              />
            </div>

            <div>
              <label class="block mb-2 text-sm text-[#D8D2C8]">Nom</label>
              <input
                v-model="form.lastName"
                type="text"
                placeholder="Votre nom"
                class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
              />
            </div>
          </div>

          <div>
            <label class="block mb-2 text-sm text-[#D8D2C8]">Email</label>
            <input
              v-model="form.email"
              type="email"
              placeholder="votre@email.com"
              class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
            />
          </div>

          <div>
            <label class="block mb-2 text-sm text-[#D8D2C8]">Confirmer l'email</label>
            <input
              v-model="form.confirmEmail"
              type="email"
              placeholder="Confirmez votre email"
              class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
            />
          </div>

          <div>
            <label class="block mb-2 text-sm text-[#D8D2C8]">Mot de passe</label>
            <input
              v-model="form.password"
              type="password"
              placeholder="••••••••"
              class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
            />
          </div>

          <div>
            <label class="block mb-2 text-sm text-[#D8D2C8]">Confirmer le mot de passe</label>
            <input
              v-model="form.confirmPassword"
              type="password"
              placeholder="••••••••"
              class="w-full rounded-2xl border border-[#2A2A2E] bg-[#121214] px-4 py-4 text-[#F5F1E8] outline-none focus:border-[#E7DDD0]"
            />
          </div>

          <p v-if="errorMessage" class="text-red-400 text-sm">
            {{ errorMessage }}
          </p>

          <p v-if="successMessage" class="text-green-400 text-sm">
            {{ successMessage }}
          </p>

          <button
            @click="handleRegister"
            :disabled="loading"
            class="w-full rounded-2xl bg-[#E7DDD0] px-4 py-4 font-semibold text-[#111112] transition hover:opacity-90 disabled:opacity-60"
          >
            {{ loading ? 'Création en cours...' : 'Créer mon compte' }}
          </button>

          <p class="text-sm text-[#B9B3A8]">
            Vous avez déjà un compte ?
            <router-link to="/login" class="text-[#E7DDD0] hover:underline">
              Se connecter
            </router-link>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>