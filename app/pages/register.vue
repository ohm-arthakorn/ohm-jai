<!-- ส่วนของหน้าสมัครสมาชิก -->
<template>
  <div class="flex flex-col items-center justify-center min-h-screen bg-gray-50 p-6">
    <div class="w-full max-w-md bg-white rounded-lg shadow-md p-8">

      <!-- ส่วนของ header -->
      <h1 class="text-3xl font-bold text-center py-2">📝</h1>
      <h2 class="text-2xl font-bold text-center mb-8 text-gray-800">สมัครบัญชีใหม่</h2>

      <form @submit.prevent="handleRegister">
        <div class="space-y-4">
          <!-- ช่องกรอกข้อมูลต่างๆ -->
          <input v-model="username" type="text" placeholder="Username"
            class="w-full p-3 border rounded focus:ring focus:ring-blue-200 outline-none transition-all" required />
          <input v-model="password" type="password" placeholder="Password"
            class="w-full p-3 border rounded focus:ring focus:ring-blue-200 outline-none transition-all" required />
          <input v-model="confirmPassword" type="password" placeholder="Confirm Password"
            class="w-full p-3 border rounded focus:ring focus:ring-blue-200 outline-none transition-all" required />

          <!-- ข้อความตัวหนังสือสีแดงและเขียวสำหรับแจ้งเตือนสถานะ -->
          <p v-if="errorMessage" class="text-red-500 text-sm text-center font-medium">{{ errorMessage }}</p>
          <p v-if="successMessage" class="text-green-600 text-sm text-center font-bold">{{ successMessage }}</p>

          <!-- ปุ่มดำเนินการ -->
          <button type="submit" :disabled="loading"
            class="w-full bg-green-600 text-white p-3 rounded font-bold hover:bg-green-700 disabled:opacity-50 transition-colors">
            {{ loading ? 'กำลังสร้างบัญชี...' : 'ยืนยันในการสมัครสมาชิก' }}
          </button>

          <!-- ส่วนเชื่อมกลับไปยังหน้าล็อกอิน -->
          <div class="text-center mt-6">
            <span class="text-gray-600 text-sm">มีบัญชีผู้ใช้งานแล้วใช่ไหม? </span>
            <NuxtLink to="/login" class="text-blue-600 hover:underline text-sm font-bold transition-all">
              กลับสู่หน้าเข้าสู่ระบบ</NuxtLink>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
// ปิดไม่ให้ใช้งาน Navbar ร่วมกับที่เหลือ
definePageMeta({
  layout: false
})

const username = ref('')
const password = ref('')
const confirmPassword = ref('')
const errorMessage = ref('')
const successMessage = ref('')
const loading = ref(false)

const handleRegister = async () => {
  // 1. ตรวจสอบว่ากรอกครบไหม
  if (!username.value || !password.value || !confirmPassword.value) {
    errorMessage.value = 'กรุณากรอกข้อมูลในช่องให้ครบทุกช่อง'
    return
  }

  // 2. ตรวจสอบว่ายืนยันรหัสผ่านตรงกับรหัสผ่านเดิมหรือไม่
  if (password.value !== confirmPassword.value) {
    errorMessage.value = 'รหัสผ่านและการยืนยันรหัสผ่านไม่ตรงกัน !'
    return
  }

  loading.value = true
  errorMessage.value = ''
  successMessage.value = ''

  try {
    // ส่งข้อมูลแบบ POST ไปที่ตัว Backend ที่โอมมีอยู่แล้ว
    const data = await $fetch('/api/auth/register', {
      method: 'POST',
      body: {
        username: username.value,
        password: password.value
      }
    })

    // ถ้าสร้างสำเร็จ
    if (data.success) {
      successMessage.value = '🎉 สมัครสมาชิกสำเร็จ! ระบบกำลังนำคุณกลับไปหน้าล็อกอิน...'
      // เมื่อสมัครสำเร็จ หน่วงเวลา 2 วินาที แล้วจับผู้ใช้ย้ายหน้า
      setTimeout(() => {
        navigateTo('/login')
      }, 2000)
    }
  } catch (error: any) {
    // ถ้าฝั่ง Backend มีการโยน P2002 ตัวหนังสือ "ชื่อนี้มีผู้ใช้งานแล้ว" มาให้ จะรับได้จากตรงนี้
    if (error.response?._data?.statusMessage) {
      errorMessage.value = error.response._data.statusMessage
    } else {
      errorMessage.value = 'ระบบขัดข้อง ไม่สามารถสมัครสมาชิกได้ในขณะนี้'
    }
  } finally {
    loading.value = false
  }
}
</script>
