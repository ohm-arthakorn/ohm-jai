<template>
    <div class="p-6 max-w-2xl mx-auto mt-10 space-y-6">
        <div class="bg-white p-6 rounded-2xl shadow-xl space-y-4">
            <h1 class="text-2xl font-bold text-gray-800 border-b pb-2">ทดสอบการเชื่อมต่อ Supabase 🚀</h1>

            <!-- สถานะการดึงข้อมูล -->
            <div v-if="pending" class="text-blue-500 animate-pulse font-medium">
                กำลังโหลดข้อมูล...
            </div>
            <div v-else-if="error" class="text-red-500 bg-red-50 p-4 rounded-xl border border-red-200">
                <p class="font-bold">เกิดข้อผิดพลาด!</p>
                <p class="text-sm mt-1">{{ error.message }}</p>
            </div>

            <!-- แสดงข้อมูลที่ดึงได้ (ถ้ามี) -->
            <div v-else-if="transactions && transactions.length > 0" class="space-y-4">
                <p class="text-green-600 font-medium bg-green-50 p-3 rounded-lg border border-green-200">
                    ✅ เชื่อมต่อและดึงข้อมูลสำเร็จ! พบ {{ transactions.length }} รายการ
                </p>

                <div class="bg-gray-50 rounded-xl overflow-hidden border border-gray-100">
                    <ul class="divide-y divide-gray-200">
                        <li v-for="tx in transactions" :key="tx.id" class="p-4 hover:bg-white transition-colors">
                            <div class="flex justify-between items-start">
                                <div>
                                    <p class="font-semibold text-gray-800">{{ tx.title }}</p>
                                    <p class="text-xs text-gray-500 mt-1">{{ tx.category }} • {{
                                        formatDate(tx.created_at) }}</p>
                                </div>
                                <div class="text-right">
                                    <p :class="tx.type === 'income' ? 'text-green-500' : 'text-red-500'"
                                        class="font-bold">
                                        {{ tx.type === 'income' ? '+' : '-' }}฿{{ tx.amount }}
                                    </p>
                                    <span
                                        class="text-[10px] px-2 py-1 rounded-full bg-gray-200 text-gray-600 mt-2 inline-block">
                                        {{ tx.type }}
                                    </span>
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>

            <!-- กรณีที่เชื่อมต่อได้ แต่ยังไม่มีข้อมูล -->
            <div v-else class="text-yellow-600 bg-yellow-50 p-4 rounded-xl border border-yellow-200">
                <p class="font-bold">เชื่อมต่อสำเร็จ แต่ยังไม่มีข้อมูล 📝</p>
                <p class="text-sm mt-1">ลองไปเพิ่มข้อมูลในตาราง `transactions` บนเว็บ Supabase ดูนะครับ</p>
            </div>
        </div>

        <div class="text-center">
            <NuxtLink to="/" class="text-blue-500 hover:text-blue-700 text-sm font-medium transition-colors">
                กลับไปหน้าหลัก
            </NuxtLink>
        </div>
    </div>
</template>

<script setup>
// กำหนดให้หน้านี้ไม่ต้องใช้ Layout หลัก เพื่อให้ดูเป็นหน้าทดสอบแยกต่างหากชัดเจน
definePageMeta({
    layout: false
})

const supabase = useSupabaseClient()

// ดึงข้อมูลจากตาราง transactions
const { data: transactions, pending, error } = await useAsyncData('test-transactions', async () => {
    const { data, error } = await supabase
        .from('transactions')
        .select('*')
        .order('created_at', { ascending: false })
        .limit(5) // ดึงมาเทสแค่ 5 รายการล่าสุด

    if (error) throw error
    return data
})

// ฟังก์ชันสำหรับแปลงรูปแบบวันที่ให้น่าอ่าน
const formatDate = (dateString) => {
    if (!dateString) return 'ไม่มีวันที่'
    const date = new Date(dateString)
    return new Intl.DateTimeFormat('th-TH', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
    }).format(date)
}
</script>
