<template>
    <div class="flex flex-col h-full gap-6">
        <header>
            <h1 class="text-2xl font-bold text-slate-800">บันทึกรายการ</h1>
        </header>

        <div class="bg-white rounded-2xl shadow-sm border border-slate-100 p-1 flex mb-2">
            <button @click="form.type = 'expense'"
                class="flex-1 py-2 text-sm font-medium rounded-xl transition-colors disabled:opacity-50"
                :class="form.type === 'expense' ? 'bg-red-50 text-red-600' : 'text-slate-500 hover:bg-slate-50'">
                รายจ่าย
            </button>
            <button @click="form.type = 'income'"
                class="flex-1 py-2 text-sm font-medium rounded-xl transition-colors disabled:opacity-50"
                :class="form.type === 'income' ? 'bg-emerald-50 text-emerald-600' : 'text-slate-500 hover:bg-slate-50'">
                รายรับ
            </button>
        </div>

        <form @submit.prevent="saveTransaction" class="flex flex-col gap-5 flex-1 pb-10">
            <!-- Amount -->
            <div class="flex flex-col gap-2">
                <label class="text-xs font-semibold text-slate-500 uppercase tracking-wider">จำนวนเงิน (บาท)</label>
                <div class="relative">
                    <span class="absolute left-4 top-1/2 -translate-y-1/2 text-slate-400 font-bold text-xl">฿</span>
                    <input v-model="form.amount" type="number" placeholder="0.00" required
                        class="w-full pl-10 pr-4 py-4 bg-white border border-slate-200 rounded-xl text-2xl font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-shadow" />
                </div>
            </div>

            <!-- Category -->
            <div class="flex flex-col gap-2">
                <label class="text-xs font-semibold text-slate-500 uppercase tracking-wider">หมวดหมู่</label>
                <select v-model="form.category" required
                    class="w-full px-4 py-3 border border-slate-200 rounded-xl text-slate-800 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent appearance-none bg-white">
                    <option value="" disabled>เลือกหมวดหมู่</option>
                    <option v-for="cat in availableCategories" :key="cat" :value="cat">{{ cat }}</option>
                </select>
            </div>

            <!-- Note -->
            <div class="flex flex-col gap-2">
                <label class="text-xs font-semibold text-slate-500 uppercase tracking-wider">จดบันทึก (ตัวเลือก)</label>
                <input v-model="form.note" type="text" placeholder="เช่น ค่าข้าวต้มกุ๊ย"
                    class="w-full px-4 py-3 bg-white border border-slate-200 rounded-xl text-slate-800 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-shadow" />
            </div>

            <!-- Submit -->
            <button type="submit" :disabled="isSubmitting"
                class="mt-auto w-full py-4 rounded-xl text-white font-bold text-lg shadow-lg transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50"
                :class="form.type === 'expense' ? 'bg-red-500 hover:bg-red-600 shadow-red-200 focus:ring-red-500' : 'bg-emerald-500 hover:bg-emerald-600 shadow-emerald-200 focus:ring-emerald-500'">
                {{ isSubmitting ? 'กำลังบันทึก...' : 'บันทึกรายการ' }}
            </button>
        </form>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'nuxt/app'
import { useTransactionStore } from '~/stores/useTransactionStore'

const router = useRouter()
// เรียกใช้งาน Supabase ในเว็บไซต์
const supabase = useSupabaseClient()
const store = useTransactionStore()

const isSubmitting = ref(false)

const form = ref({
    type: 'expense',
    amount: '',
    category: '',
    note: ''
})

const expenseCategories = ['อาหาร', 'เดินทาง', 'ช้อปปิ้ง', 'บิล/ค่าใช้จ่าย', 'สุขภาพ', 'อื่นๆ']
const incomeCategories = ['เงินเดือน', 'ธุรกิจ', 'โบนัส', 'อื่นๆ']

const availableCategories = computed(() => {
    return form.value.type === 'expense' ? expenseCategories : incomeCategories
})

const saveTransaction = async () => {
    if (!form.value.amount || !form.value.category) return

    isSubmitting.value = true

    try {
        // 1. บันทึกลง Supabase
        const { data, error } = await supabase
            .from('transactions')
            .insert([
                {
                    title: form.value.note || form.value.category, // ถ้าไม่ได้จด note ให้เอา category มาใช้แทน
                    amount: Number(form.value.amount),
                    type: form.value.type,
                    category: form.value.category
                }
            ])
            .select() // เพื่อให้มันรีเทิร์นข้อมูลที่เพิ่ง insert กลับมา

        if (error) {
            console.error('Error inserting data:', error)
            alert('เกิดข้อผิดพลาดในการบันทึกข้อมูล: ' + error.message)
            return
        }

        console.log('Successfully inserted data:', data)

        // 2. อัปเดต State (Pinia) ต่อ เผื่อเอาไปใช้โชว์ที่อื่นแบบไม่ต้องโหลตใหม่
        store.addTransaction({
            id: data[0].id, // ใช้ id จาก Supabase
            type: form.value.type,
            amount: Number(form.value.amount),
            category: form.value.category,
            note: form.value.note,
            created_at: data[0].created_at // ใช้เวลาจาก Supabase
        })

        // กลับไปหน้าแรก
        router.push('/')
    } catch (err) {
        console.error('Unexpected error:', err)
        alert('เกิดข้อผิดพลาดที่ไม่คาดคิด!')
    } finally {
        isSubmitting.value = false
    }
}
</script>
