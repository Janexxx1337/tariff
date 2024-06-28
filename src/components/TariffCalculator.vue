<template>
  <div class="tariff-calculator container">
    <h1 class="my-4 text-center">Тарифный калькулятор</h1>
    <form class="border p-4 rounded bg-light shadow-sm">
      <div class="mb-3">
        <label for="tariff" class="form-label">Выбор тарифа:</label>
        <select v-model="tariff" @change="needRecalculate = true" id="tariff" class="form-select">
          <option value="standard">Стандартный</option>
          <option value="advanced">Продвинутый</option>
        </select>
      </div>

      <div class="mb-3">
        <label for="currency" class="form-label">Выбор валюты:</label>
        <select v-model="currency" id="currency" class="form-select">
          <option value="rub">Рубль</option>
          <option value="cny">Китайский юань</option>
          <option value="kzt">Казахстанский тенге</option>
        </select>
      </div>

      <div class="mb-3">
        <label class="form-label">Выбор периода оплаты:</label>
        <div class="form-check">
          <input type="radio" id="monthly" value="monthly" v-model="period" @change="needRecalculate = true" class="form-check-input" />
          <label for="monthly" class="form-check-label">За месяц</label>
        </div>
        <div class="form-check">
          <input type="radio" id="yearly" value="yearly" v-model="period" @change="needRecalculate = true" class="form-check-input" />
          <label for="yearly" class="form-check-label">За год</label>
        </div>
      </div>

      <button type="button" @click="calculateTotal" class="btn btn-primary w-100">Рассчитать</button>
    </form>

    <div v-if="needRecalculate" class="mt-4 text-center text-warning">
      <p>Пожалуйста, нажмите кнопку "Рассчитать" для обновления суммы.</p>
    </div>

    <div v-if="total !== null" class="mt-4 text-center">
      <p class="h5">Сумма для оплаты: <span class="fw-bold">{{ total }}</span> {{ currency }}</p>
      <p v-if="discount" class="text-success">Сумма скидки: <span class="fw-bold">{{ discount }}</span> {{ currency }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'

const tariff = ref('standard')
const currency = ref('rub')
const period = ref('monthly')
const total = ref(null)
const discount = ref(null)

const exchangeRates = ref({
  rub: 1,
  cny: 1,
  kzt: 1
})
const needRecalculate = ref(false)

const calculateTotal = () => {
  const prices = {
    standard: { monthly: 100, yearly: 1000 },
    advanced: { monthly: 150, yearly: 1400 }
  }

  const selectedTariff = prices[tariff.value]
  const price = selectedTariff[period.value]
  const rate = exchangeRates.value[currency.value]

  total.value = (price * rate).toFixed(2)
  discount.value = period.value === 'yearly' ? ((selectedTariff.monthly * 12 - price) * rate).toFixed(2) : null
  needRecalculate.value = false
}

const fetchExchangeRates = async () => {
  try {
    const response = await axios.get('https://api.exchangerate-api.com/v4/latest/RUB')
    exchangeRates.value.cny = response.data.rates.CNY
    exchangeRates.value.kzt = response.data.rates.KZT
  } catch (error) {
    console.error('Error fetching exchange rates:', error)
  }
}

onMounted(() => {
  fetchExchangeRates()
})

// Добавляем watch для отслеживания изменений в currency и вызываем calculateTotal при изменении
watch(currency, () => {
  calculateTotal()
})
</script>

<style scoped>
.tariff-calculator {
  max-width: 500px;
  margin: 0 auto;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.form-label {
  font-weight: bold;
}

select, input[type="radio"] {
  margin-top: 10px;
}

button {
  margin-top: 20px;
}

h1 {
  color: #333;
}

.tariff-calculator {
  padding: 15px;
  margin-top: 20px;
}

.text-warning {
  color: #ffc107;
}
</style>
