<!-- <script setup>
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style> -->


<template>
  <div class="container">
    <h1>Регистрация</h1>
    
    <form @submit.prevent="handleSubmit" novalidate>
      <!-- Имя (2-50) -->
      <div class="form-group">
        <label for="firstName">Имя*</label>
        <input
          id="firstName"
          v-model.trim="formData.firstName"
          type="text"
          required
          minlength="2"
          maxlength="50"
          @blur="markTouched('firstName')"
          :class="{ error: touched.firstName && errors.firstName }"
        />
        <div v-if="touched.firstName && errors.firstName" class="error-message">
          {{ errors.firstName }}
        </div>
      </div>

      <!-- E-mail с асинхронной проверкой -->
      <div class="form-group">
        <label for="email">E-mail*</label>
        <input
          id="email"
          v-model.trim="formData.email"
          type="email"
          required
          @blur="markTouched('email')"
          @input="handleEmailInput"
          :class="{ error: touched.email && errors.email }"
        />
        <div v-if="emailBusy" class="info-message">Проверяем доступность email...</div>
        <div v-else-if="touched.email && errors.email" class="error-message">
          {{ errors.email }}
        </div>
      </div>

      <!-- Возраст (14-120, число) -->
      <div class="form-group">
        <label for="age">Возраст*</label>
        <input
          id="age"
          v-model.number="formData.age"
          type="number"
          required
          min="14"
          max="120"
          @blur="markTouched('age')"
          :class="{ error: touched.age && errors.age }"
        />
        <div v-if="touched.age && errors.age" class="error-message">
          {{ errors.age }}
        </div>
      </div>

      <!-- Пароль (мин 8, ≥1 цифра) -->
      <div class="form-group">
        <label for="password">Пароль*</label>
        <input
          id="password"
          v-model.lazy="formData.password"
          type="password"
          required
          minlength="8"
          @blur="markTouched('password')"
          :class="{ error: touched.password && errors.password }"
        />
        <div v-if="touched.password && errors.password" class="error-message">
          {{ errors.password }}
        </div>
      </div>

      <!-- Повтор пароля -->
      <div class="form-group">
        <label for="confirmPassword">Повтор пароля*</label>
        <input
          id="confirmPassword"
          v-model.lazy="formData.confirmPassword"
          type="password"
          required
          @blur="markTouched('confirmPassword')"
          :class="{ error: touched.confirmPassword && errors.confirmPassword }"
        />
        <div v-if="touched.confirmPassword && errors.confirmPassword" class="error-message">
          {{ errors.confirmPassword }}
        </div>
      </div>

      <!-- Тип аккаунта (student|business) -->
      <div class="form-group">
        <label>Тип аккаунта*</label>
        <div class="radio-group">
          <label>
            <input
              type="radio"
              v-model="formData.accountType"
              value="student"
              @change="markTouched('accountType')"
            />
            Student
          </label>
          <label>
            <input
              type="radio"
              v-model="formData.accountType"
              value="business"
              @change="markTouched('accountType')"
            />
            Business
          </label>
        </div>
        <div v-if="touched.accountType && errors.accountType" class="error-message">
          {{ errors.accountType }}
        </div>
      </div>

      <!-- Согласие -->
      <div class="form-group">
        <label class="checkbox-label">
          <input
            type="checkbox"
            v-model="formData.agree"
            @change="markTouched('agree')"
          />
          Соглашаюсь с условиями*
        </label>
        <div v-if="touched.agree && errors.agree" class="error-message">
          {{ errors.agree }}
        </div>
      </div>

      <!-- Кнопка отправки -->
      <button 
        type="submit" 
        :disabled="!isValid || isSubmitting"
        class="submit-btn"
      >
        {{ isSubmitting ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>
  </div>
</template>

<script setup>
import { reactive, computed, ref } from 'vue'

// Данные формы (values)
const formData = reactive({
  firstName: '',
  email: '',
  age: null,
  password: '',
  confirmPassword: '',
  accountType: '',
  agree: false
})

// Состояния touched
const touched = reactive({
  firstName: false,
  email: false,
  age: false,
  password: false,
  confirmPassword: false,
  accountType: false,
  agree: false
})

// Ошибки
const errors = reactive({
  firstName: '',
  email: '',
  age: '',
  password: '',
  confirmPassword: '',
  accountType: '',
  agree: ''
})

// UI состояния
const emailBusy = ref(false)
const isSubmitting = ref(false)

// Таймер для дебаунса
let emailTimer = null

// Валидация полей
const validateField = (fieldName) => {
  const value = formData[fieldName]
  
  switch (fieldName) {
    case 'firstName':
      if (!value) {
        errors.firstName = 'Имя обязательно'
      } else if (value.length < 2 || value.length > 50) {
        errors.firstName = 'Имя должно быть от 2 до 50 символов'
      } else {
        errors.firstName = ''
      }
      break
      
    case 'email':
      if (!value) {
        errors.email = 'Email обязателен'
      } else if (!/^\S+@\S+\.\S+$/.test(value)) {
        errors.email = 'Неверный формат email'
      } else {
        // Асинхронная проверка будет в handleEmailInput
        errors.email = ''
      }
      break
      
    case 'age':
      if (value === null || value === '') {
        errors.age = 'Возраст обязателен'
      } else if (typeof value !== 'number' || isNaN(value)) {
        errors.age = 'Возраст должен быть числом'
      } else if (value < 14 || value > 120) {
        errors.age = 'Возраст должен быть от 14 до 120 лет'
      } else {
        errors.age = ''
      }
      break
      
    case 'password':
      if (!value) {
        errors.password = 'Пароль обязателен'
      } else if (value.length < 8) {
        errors.password = 'Пароль должен содержать минимум 8 символов'
      } else if (!/\d/.test(value)) {
        errors.password = 'Пароль должен содержать хотя бы одну цифру'
      } else {
        errors.password = ''
      }
      break
      
    case 'confirmPassword':
      if (!value) {
        errors.confirmPassword = 'Повтор пароля обязателен'
      } else if (value !== formData.password) {
        errors.confirmPassword = 'Пароли не совпадают'
      } else {
        errors.confirmPassword = ''
      }
      break
      
    case 'accountType':
      if (!value) {
        errors.accountType = 'Выберите тип аккаунта'
      } else {
        errors.accountType = ''
      }
      break
      
    case 'agree':
      if (!value) {
        errors.agree = 'Необходимо согласие с условиями'
      } else {
        errors.agree = ''
      }
      break
  }
}

// Асинхронная проверка email (фейковая)
const checkEmailAvailability = async (email) => {
  // Имитация запроса к серверу
  return new Promise((resolve) => {
    setTimeout(() => {
      // Email занят если домен @test.com
      const isAvailable = !email.endsWith('@test.com')
      resolve(isAvailable)
    }, 500)
  })
}

// Обработчик ввода email с дебаунсом
const handleEmailInput = () => {
  if (touched.email) {
    validateField('email')
  }
  
  // Дебаунс 500ms
  clearTimeout(emailTimer)
  emailTimer = setTimeout(async () => {
    if (formData.email && /^\S+@\S+\.\S+$/.test(formData.email)) {
      emailBusy.value = true
      const isAvailable = await checkEmailAvailability(formData.email)
      emailBusy.value = false
      
      if (!isAvailable) {
        errors.email = 'Этот email уже занят'
      } else if (touched.email) {
        errors.email = ''
      }
    }
  }, 500)
}

// Пометить поле как touched
const markTouched = (fieldName) => {
  touched[fieldName] = true
  validateField(fieldName)
  
  // Особые случаи
  if (fieldName === 'password' || fieldName === 'confirmPassword') {
    validateField('password')
    validateField('confirmPassword')
  }
}

// Валидация всей формы
const validateAll = () => {
  Object.keys(touched).forEach(key => {
    touched[key] = true
    validateField(key)
  })
}

// Фокус на первую ошибку
const focusFirstError = () => {
  const firstErrorField = Object.keys(errors).find(key => errors[key])
  if (firstErrorField) {
    const element = document.getElementById(firstErrorField)
    if (element) {
      element.focus()
    }
  }
}

// Вычисляемое свойство isValid
const isValid = computed(() => {
  return !Object.values(errors).some(error => error) &&
    formData.firstName &&
    formData.email &&
    formData.age !== null &&
    formData.password &&
    formData.confirmPassword &&
    formData.accountType &&
    formData.agree
})

// Обработчик отправки формы
const handleSubmit = async () => {
  validateAll()
  
  if (!isValid.value) {
    focusFirstError()
    return
  }
  
  isSubmitting.value = true
  
  try {
    // Имитация отправки на сервер
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    alert('Регистрация успешно завершена!')
    console.log('Данные формы:', formData)
    
    // Сброс формы
    Object.assign(formData, {
      firstName: '',
      email: '',
      age: null,
      password: '',
      confirmPassword: '',
      accountType: '',
      agree: false
    })
    
    Object.keys(touched).forEach(key => {
      touched[key] = false
    })
    
  } catch (error) {
    alert('Ошибка при регистрации')
  } finally {
    isSubmitting.value = false
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

body {
  background-color: #f5f5f5;
  margin: 0;
  padding: 20px;
}

.container {
  max-width: 500px;
  margin: 0 auto;
  background: white;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

input[type="text"],
input[type="email"],
input[type="number"],
input[type="password"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

input.error {
  border-color: #ff4444;
}

.radio-group {
  display: flex;
  gap: 20px;
  margin-top: 5px;
}

.radio-group label {
  display: flex;
  align-items: center;
  font-weight: normal;
}

.radio-group input {
  margin-right: 5px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  font-weight: normal;
}

.checkbox-label input {
  margin-right: 8px;
}

.error-message {
  color: #ff4444;
  font-size: 14px;
  margin-top: 5px;
}

.info-message {
  color: #666;
  font-size: 14px;
  margin-top: 5px;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
}

.submit-btn:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

.submit-btn:hover:not(:disabled) {
  background-color: #0056b3;
}
</style>
