<template>
  <q-page padding>
    <div class="q-pa-md">
      <h2 class="text-center text-primary">Previsão do Tempo</h2>
    </div>

    <q-input
      v-model="city"
      label="Digite o nome da cidade"
      @keyup.enter="fetchWeather"
      autofocus
      class="q-mb-md"
    />
    <q-spinner v-if="loading" size="50px" color="primary" class="q-my-md" />

    <div v-if="error" class="text-center text-negative q-my-md">{{ error }}</div>
    
    <q-layout view="lHh Lpr lFf">
      <q-page-container>
        <q-page class="q-pa-none">
          <q-card-group class="q-mb-md" style="display: flex; flex-wrap: wrap; justify-content: space-evenly;">
            <div v-for="(weather, index) in weatherResults" :key="index" class="q-pa-md">
              <q-card :class="getCardClass(weather.condition.text)" style="max-width: 320px; margin-bottom: 20px;">
                <q-card-section>
                  <div class="text-center">
                    <q-img :src="weather.icon" alt="Ícone do clima" class="weather-icon" />
                    <h6 class="q-mt-md text-primary">Temperatura: {{ roundTemperature(weather.temp_c) }}°C</h6>
                    <p class="q-mb-md text-primary">{{ translateCondition(weather.condition.text) }}</p>
                    <p class="q-mb-md text-primary">{{ weather.city }}, {{ weather.state }}, {{ weather.country }}</p>
                    <p class="q-mb-md text-primary">Hora local: {{ weather.localTime }}</p>
                    <p v-if="weather.condition.text.toLowerCase().includes('fog')" class="q-mb-md text-negative">Aviso: Neblina detectada! Visibilidade reduzida.</p>
                  </div>
                </q-card-section>

                <q-card-actions>
                  <q-btn label="Detalhes" color="primary" @click="showDetails(weather)" />
                </q-card-actions>
              </q-card>
            </div>
          </q-card-group>
        </q-page>
      </q-page-container>
    </q-layout>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

// Definindo variáveis reativas
const city = ref('')
const weatherResults = ref([])
const error = ref('')
const loading = ref(false)

// Pegando as variáveis de ambiente para a chave da API e o URL
const apiKey = import.meta.env.VITE_WEATHER_API_KEY
const apiUrl = import.meta.env.VITE_WEATHER_API_URL

// Função para buscar o clima
const fetchWeather = async () => {
  if (!city.value) {
    error.value = 'Por favor, digite o nome de uma cidade.'
    return
  }

  loading.value = true
  error.value = ''

  try {
    const response = await axios.get(`${apiUrl}/current.json?key=${apiKey}&q=${city.value}`)
    const data = response.data.current
    const location = response.data.location // Informações de localização

    const newWeather = {
      temp_c: data.temp_c,
      condition: data.condition,
      humidity: data.humidity,
      wind_kph: data.wind_kph,
      icon: `http:${data.condition.icon}`,
      city: location.name,
      state: location.region,
      country: location.country,
      localTime: formatTime(location.localtime),
    }

    weatherResults.value.unshift(newWeather)
  } catch {
    error.value = 'Erro ao buscar dados do clima'
  } finally {
    loading.value = false
  }
}

// Função para formatar a hora local
const formatTime = (localtime) => {
  const date = new Date(localtime)
  const hours = date.getHours().toString().padStart(2, '0')
  const minutes = date.getMinutes().toString().padStart(2, '0')
  return `${hours}:${minutes}`
}

const showDetails = (weather) => {
  alert(`Detalhes do clima: \nTemperatura: ${weather.temp_c}°C\nVento: ${weather.wind_kph} km/h\nUmidade: ${weather.humidity}%`)
}

const getCardClass = (conditionText) => {
  if (conditionText.toLowerCase().includes('rain')) {
    return 'rainy-card'
  } else if (conditionText.toLowerCase().includes('cloudy')) {
    return 'cloudy-card'
  } else if (conditionText.toLowerCase().includes('sunny')) {
    return 'sunny-card'
  } else if (conditionText.toLowerCase().includes('fog')) {
    return 'foggy-card'
  }
  return ''
}

const translateCondition = (condition) => {
  if (condition.toLowerCase() === 'clear') {
    return 'Céu Limpo'
  } else if (condition.toLowerCase() === 'patchy light drizzle') {
    return 'Chuvisco Leve'
  } else if (condition.toLowerCase() === 'partly cloudy') {
    return 'Parcialmente Nublado'
  } else if (condition.toLowerCase() === 'cloudy') {
    return 'Nublado'
  } else if (condition.toLowerCase() === 'sunny') {
    return 'Ensolarado'
  } else if (condition.toLowerCase() === 'fog') {
    return 'Nebuloso'
  }
  return condition
}

const roundTemperature = (temp) => {
  return Math.round(temp)
}
</script>

<style scoped>
/* Estilo da página */
.q-page {
  background-color: #f5f5f5;
}
.weather-icon {
  width: 80px;
  height: 80px;
}
.text-negative {
  color: #d9534f;
}
.text-primary {
  color: #007bff;
}
.rainy-card {
  background-color: #c6c6c6;
}
.cloudy-card {
  background-color: #f0f0f0;
}
.sunny-card {
  background-color: #ffeb3b;
}
.foggy-card {
  background-color: #bdbdbd;
}
.q-card-section {
  padding: 16px;
}
.q-btn {
  width: 100%;
}
.q-spinner {
  margin-top: 20px;
}
.q-pa-md {
  padding: 16px;
}
</style>
