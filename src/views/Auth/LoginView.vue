<template>
  <div class="login-page">
    <div class="login-card">

      <!-- Header dengan logo -->
      <div class="login-header">
        <h1>🎬 CineVue</h1>
        <h2>Login Admin</h2>
      </div>

      <!-- Alert error: hanya tampil jika ada pesan error -->
      <div v-if="errorMsg" class="alert alert-error">
        ❌ {{ errorMsg }}
      </div>

      <!--
        @submit.prevent="handleLogin"
        ↑ Dua hal di sini:
        1. @submit = dengarkan event "submit" form
        2. .prevent = cegah behavior default browser (reload halaman)
        3. handleLogin = fungsi yang dijalankan saat form disubmit
      -->
      <form @submit.prevent="handleLogin" class="login-form">

        <div class="form-group">
          <label for="email">📧 Email</label>
          <!--
            v-model="form.email"
            → Two-way binding: input ↔ form.email
            → Ketik di input → form.email berubah otomatis
            → form.email berubah → input berubah otomatis
          -->
          <input
            id="email"
            v-model="form.email"
            type="email"
            placeholder="admin@example.com"
            required
          />
        </div>

        <div class="form-group">
          <label for="password">🔑 Password</label>
          <input
            id="password"
            v-model="form.password"
            type="password"
            placeholder="••••••••"
            required
          />
        </div>

        <!--
          :disabled="loading"
          → Tombol tidak bisa diklik saat loading = true
          → Mencegah user klik berkali-kali saat menunggu respons API
        -->
        <button type="submit" :disabled="loading" class="btn-login">
          <span v-if="loading">⏳ Memproses...</span>
          <span v-else>🚀 Login</span>
        </button>

      </form>

      <div class="login-footer">
        <RouterLink to="/">← Kembali ke Home</RouterLink>
      </div>

    </div>
  </div>
</template>
<script setup>
// ─── STEP 1: Import ──────────────────────────────────────
import { ref, reactive, onMounted } from 'vue'
import { useRouter, RouterLink }    from 'vue-router'
import axios                        from 'axios'

// useRouter() → untuk berpindah halaman secara programatik
// (berbeda dengan RouterLink yang di template)
const router = useRouter()

// ─── STEP 2: Form menggunakan reactive() ─────────────────
// Kenapa reactive()? Karena kita punya 2+ field yang saling terkait
const form = reactive({
  email:    '',
  password: ''
})

// Variabel status menggunakan ref()
const loading  = ref(false)  // status tombol login
const errorMsg = ref('')     // pesan error login

const BASE_URL = '<http://localhost:8000/api>'

// ─── STEP 3: Auto-redirect jika sudah login ──────────────
// Jika user sudah punya token, langsung masuk dashboard
// (tidak perlu login lagi)
onMounted(() => {
  if (localStorage.getItem('token')) {
    router.push('/dashboard')
  }
})

// ─── STEP 4: Fungsi Login ─────────────────────────────────
const handleLogin = async () => {
  try {
        // Kirim POST request ke endpoint login
    const response = await axios.post(`${BASE_URL}/login`, {
      email:    form.email,
      password: form.password,
    })

    /*
      Struktur response dari API kita:
      {
        "token": "5|abc123xyz456..."
      }
      ⚠️ Catatan: API kita HANYA mengembalikan token, tidak ada "user"!
      Kita perlu satu request lagi ke /profile untuk ambil data user.
    */
    const token = response.data.token

    // Step 1: Simpan token terlebih dahulu
    localStorage.setItem('token', token)

    // Step 2: Ambil data user dari /profile menggunakan token yang baru dapat
    // (API login tidak return data user, jadi harus GET /profile secara terpisah)
    const profileRes = await axios.get(`${BASE_URL}/profile`, {
      headers: { Authorization: `Bearer ${token}` }
    })
    // Struktur response /profile: { data: { id, name, email, ... } }
    const user = profileRes.data.data

    // Step 3: Simpan data user ke localStorage
    localStorage.setItem('user', JSON.stringify(user))

    // Step 4: Baru redirect ke dashboard
    router.push('/dashboard')

  } catch (err) {
    /*
      HTTP Status Code yang umum dari API:
      401 → Unauthorized (email/password salah)
      422 → Unprocessable Entity (validasi gagal, format salah)
      500 → Internal Server Error (masalah di server)
    */
       if (err.response?.status === 404) {
      errorMsg.value = 'Email tidak terdaftar!'
    } else if (err.response?.status === 401) {
      errorMsg.value = 'Password salah!'
    } else if (err.response?.status === 422) {
      errorMsg.value = 'Format email tidak valid!'
    } else {
      errorMsg.value = 'Terjadi kesalahan. Pastikan server Laravel berjalan!'
    }
  } finally {
    loading.value = false
  }
}
</script>
<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  padding: 24px;
}

.login-card {
  background: white;
  padding: 48px 40px;
  border-radius: 20px;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.4);
}

.login-header {
  text-align: center;
  margin-bottom: 32px;
}

.login-header h1 {
  font-size: 28px;
  color: #e94560;
  margin-bottom: 8px;
}

.login-header h2 {
  font-size: 20px;
  color: #1a1a2e;
  margin-bottom: 6px;
}

.login-header p {
  color: #888;
  font-size: 14px;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

label {
  font-size: 13px;
  font-weight: 600;
  color: #444;
}

input {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 14px;
  font-family: inherit;
  transition: border-color 0.2s, box-shadow 0.2s;
  outline: none;
}

input:focus {
  border-color: #e94560;
  box-shadow: 0 0 0 3px rgba(233, 69, 96, 0.12);
}

.btn-login {
  margin-top: 8px;
  padding: 14px;
  background: linear-gradient(135deg, #e94560, #c73652);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 700;
  font-family: inherit;
  cursor: pointer;
  transition: opacity 0.2s, transform 0.1s;
}

.btn-login:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
}

.btn-login:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.login-footer {
  text-align: center;
  margin-top: 24px;
  font-size: 13px;
}
</style>