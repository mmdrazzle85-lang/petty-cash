<script setup>
import { ref, computed, watch } from 'vue'

const managerName = 'علی رضایی'
const budgetLimit = 10000000
const isFormVisible = ref(false)
const searchQuery = ref('')
const errorMessage = ref('')

const newExpense = ref({
  title: '',
  amount: null,
  category: 'پذیرایی',
  isPaid: false,
})

function loadExpenses() {
  const saved = localStorage.getItem('expenses')

  if (saved) {
    try {
      return JSON.parse(saved)
    } catch (error) {
      localStorage.removeItem('expenses')
    }
  }

  return [
    { id: 1, title: 'خرید چای و قند برای دفتر', amount: 350000, category: 'پذیرایی', isPaid: true },
    { id: 2, title: 'هزینه پیک مدارک اداری', amount: 120000, category: 'حمل‌ونقل', isPaid: false },
    { id: 3, title: 'خرید بسته کاغذ A4', amount: 550000, category: 'ملزومات', isPaid: true },
  ]
}

const expenses = ref(loadExpenses())

const totalExpenses = computed(() =>
  expenses.value.reduce((total, expense) => total + expense.amount, 0)
)

const remainingBudget = computed(() => budgetLimit - totalExpenses.value)

const usedPercent = computed(() =>
  Math.min(100, Math.round((totalExpenses.value / budgetLimit) * 100))
)

const filteredExpenses = computed(() =>
  expenses.value.filter((expense) => expense.title.includes(searchQuery.value))
)

function formatNumber(number) {
  return number.toLocaleString('fa-IR')
}

function addExpense() {
  const { title, amount, category, isPaid } = newExpense.value

  if (!title || !amount || amount <= 0) {
    errorMessage.value = 'عنوان و مبلغ معتبر را وارد کنید.'
    return
  }

  if (amount > remainingBudget.value) {
    errorMessage.value = 'مبلغ بیشتر از بودجه باقی مانده است.'
    return
  }

  expenses.value.push({ id: Date.now(), title, amount, category, isPaid })

  newExpense.value = { title: '', amount: null, category: 'پذیرایی', isPaid: false }
  errorMessage.value = ''
  isFormVisible.value = false
}

function toggleFullscreen() {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen()
  } else {
    document.exitFullscreen()
  }
}

watch(
  expenses,
  (newExpenses) => {
    localStorage.setItem('expenses', JSON.stringify(newExpenses))
  },
  { deep: true }
)
</script>

<template>
  <div class="relative min-h-screen w-full overflow-hidden bg-gradient-to-br from-black via-slate-950 to-blue-950">

    <!-- نورهای پس‌زمینه -->
    <div class="pointer-events-none fixed -top-32 -right-32 h-96 w-96 rounded-full bg-blue-600/30 blur-3xl"></div>
    <div class="pointer-events-none fixed top-1/2 -left-40 h-[28rem] w-[28rem] rounded-full bg-indigo-600/25 blur-3xl"></div>
    <div class="pointer-events-none fixed -bottom-32 right-1/3 h-96 w-96 rounded-full bg-sky-500/20 blur-3xl"></div>

    <div class="relative z-10 w-full space-y-6 px-4 py-6 sm:px-8 lg:px-12">

      <!-- هدر -->
      <header class="glass flex items-center justify-between p-6">
        <div class="relative">
          <h1 class="text-2xl font-bold text-white">سامانه مدیریت تنخواه</h1>
          <p class="mt-1 text-sm text-sky-200/70">نام مسئول: {{ managerName }}</p>
        </div>

        <button @click="toggleFullscreen" class="btn btn-ghost relative">
          ⛶ تمام صفحه
        </button>
      </header>

      <!-- کارت‌های بودجه -->
      <section class="grid gap-4 sm:grid-cols-3">
        <div class="glass p-5">
          <p class="relative text-sm text-slate-400">بودجه کل</p>
          <p class="relative mt-2 text-2xl font-bold text-white">
            {{ formatNumber(budgetLimit) }}
            <span class="text-sm font-normal text-slate-500">تومان</span>
          </p>
        </div>

        <div class="glass p-5">
          <p class="relative text-sm text-slate-400">جمع هزینه‌ها</p>
          <p class="relative mt-2 text-2xl font-bold text-rose-400">
            {{ formatNumber(totalExpenses) }}
            <span class="text-sm font-normal text-slate-500">تومان</span>
          </p>
        </div>

        <div class="glass p-5">
          <p class="relative text-sm text-slate-400">بودجه باقی‌مانده</p>
          <p class="relative mt-2 text-2xl font-bold text-emerald-400">
            {{ formatNumber(remainingBudget) }}
            <span class="text-sm font-normal text-slate-500">تومان</span>
          </p>
        </div>
      </section>

      <!-- نوار پیشرفت -->
      <section class="glass p-5">
        <div class="relative mb-2 flex justify-between text-sm text-slate-400">
          <span>میزان مصرف بودجه</span>
          <span>{{ usedPercent.toLocaleString('fa-IR') }}٪</span>
        </div>
        <div class="relative h-3 overflow-hidden rounded-full bg-white/10">
          <div
            class="h-full rounded-full transition-all duration-500"
            :class="usedPercent > 85
              ? 'bg-gradient-to-l from-rose-500 to-red-600'
              : usedPercent > 60
                ? 'bg-gradient-to-l from-amber-400 to-orange-500'
                : 'bg-gradient-to-l from-sky-400 to-emerald-400'"
            :style="{ width: usedPercent + '%' }"
          ></div>
        </div>
      </section>

      <!-- فرم و جست‌وجو -->
      <section class="grid gap-6 lg:grid-cols-2">

        <!-- فرم ثبت هزینه -->
        <div class="glass p-5">
          <button
            @click="isFormVisible = !isFormVisible"
            class="btn btn-primary relative"
          >
            {{ isFormVisible ? 'بستن فرم' : '+ ثبت هزینه جدید' }}
          </button>

          <form v-if="isFormVisible" @submit.prevent="addExpense" class="relative mt-5 space-y-4">
            <h3 class="text-lg font-semibold text-white">اطلاعات فاکتور</h3>

            <div>
              <label class="mb-1.5 block text-sm text-slate-300">عنوان هزینه</label>
              <input
                type="text"
                v-model.trim="newExpense.title"
                placeholder="خرید خودکار"
                class="glass-input"
              />
            </div>

            <div class="grid gap-4 sm:grid-cols-2">
              <div>
                <label class="mb-1.5 block text-sm text-slate-300">مبلغ (تومان)</label>
                <input
                  type="number"
                  v-model.number="newExpense.amount"
                  min="1"
                  placeholder="مبلغ را وارد کنید"
                  class="glass-input"
                />
              </div>

              <div>
                <label class="mb-1.5 block text-sm text-slate-300">دسته‌بندی</label>
                <select v-model="newExpense.category" class="glass-input">
                  <option value="پذیرایی">پذیرایی</option>
                  <option value="ملزومات">ملزومات</option>
                  <option value="حمل‌ونقل">حمل‌ونقل</option>
                  <option value="تعمیرات">تعمیرات</option>
                  <option value="سایر">سایر</option>
                </select>
              </div>
            </div>

            <label class="flex cursor-pointer items-center gap-2 text-sm text-slate-300">
              <input type="checkbox" v-model="newExpense.isPaid" class="h-4 w-4 accent-sky-500" />
              این فاکتور تسویه شده است
            </label>

            <p
              v-if="errorMessage"
              class="rounded-lg border border-rose-500/30 bg-rose-500/10 px-4 py-2 text-sm text-rose-300"
            >
              {{ errorMessage }}
            </p>

            <button type="submit" class="btn btn-success">ثبت فاکتور</button>
          </form>
        </div>

        <!-- جست‌وجو -->
        <div class="glass p-5">
          <h3 class="relative mb-3 text-lg font-semibold text-white">جست‌وجوی فاکتورها</h3>
          <input
            type="text"
            v-model.trim="searchQuery"
            placeholder="عنوان هزینه را بنویسید..."
            class="glass-input relative"
          />
        </div>
      </section>

      <!-- جدول -->
      <section class="glass">
        <h3 class="relative p-5 pb-3 text-lg font-semibold text-white">لیست هزینه‌ها</h3>

        <div class="relative overflow-x-auto">
          <table class="w-full text-sm">
            <thead class="bg-white/5 text-slate-300">
              <tr>
                <th class="px-5 py-3 text-start font-medium">عنوان</th>
                <th class="px-5 py-3 text-start font-medium">مبلغ</th>
                <th class="px-5 py-3 text-start font-medium">دسته‌بندی</th>
                <th class="px-5 py-3 text-start font-medium">وضعیت</th>
              </tr>
            </thead>

            <tbody class="divide-y divide-white/5">
              <tr
                v-for="expense in filteredExpenses"
                :key="expense.id"
                class="transition hover:bg-white/5"
              >
                <td class="px-5 py-3 text-white">{{ expense.title }}</td>
                <td class="whitespace-nowrap px-5 py-3 text-slate-200">
                  {{ formatNumber(expense.amount) }} تومان
                </td>
                <td class="px-5 py-3">
                  <span class="rounded-full border border-sky-400/20 bg-sky-400/10 px-3 py-1 text-xs text-sky-300">
                    {{ expense.category }}
                  </span>
                </td>
                <td class="px-5 py-3">
                  <span
                    class="rounded-full border px-3 py-1 text-xs font-medium"
                    :class="expense.isPaid
                      ? 'border-emerald-400/20 bg-emerald-400/10 text-emerald-300'
                      : 'border-rose-400/20 bg-rose-400/10 text-rose-300'"
                  >
                    {{ expense.isPaid ? 'تسویه شده' : 'تسویه نشده' }}
                  </span>
                </td>
              </tr>

              <tr v-if="filteredExpenses.length === 0">
                <td colspan="4" class="px-5 py-8 text-center text-slate-500">
                  فاکتوری پیدا نشد.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

    </div>
  </div>
</template>