<template>
  <div class="dashboard">
    <!-- Header -->
    <header class="header">
      <h1>Smart Expense Tracker</h1>
      <button @click="handleLogout" class="logout-btn">Logout</button>
    </header>

    <!-- Summary Cards -->
    <div class="summary-cards">
      <div class="card">
        <h3>Total Expenses</h3>
        <p class="amount">৳{{ totalAmount.toFixed(2) }}</p>
      </div>
      <div class="card">
        <h3>This Month</h3>
        <p class="amount">৳{{ monthlyAmount.toFixed(2) }}</p>
      </div>
      <div class="card">
        <h3>Total Records</h3>
        <p class="amount">{{ expenses.length }}</p>
      </div>
    </div>

    <!-- Add Expense Button -->
    <div class="actions">
      <button @click="showForm = true" class="add-btn">+ Add Expense</button>
    </div>

    <!-- Add Expense Form -->
    <div v-if="showForm" class="modal">
      <div class="modal-box">
        <h3>Add Expense</h3>
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label>Title</label>
            <input v-model="form.title" type="text" required />
          </div>
          <div class="form-group">
            <label>Amount (৳)</label>
            <input v-model="form.amount" type="number" step="0.01" required />
          </div>
          <div class="form-group">
            <label>Category</label>
            <select v-model="form.category" required>
              <option value="">Select category</option>
              <option value="Food">Food</option>
              <option value="Transport">Transport</option>
              <option value="Shopping">Shopping</option>
              <option value="Bills">Bills</option>
              <option value="Health">Health</option>
              <option value="Other">Other</option>
            </select>
          </div>
          <div class="form-group">
            <label>Date</label>
            <input v-model="form.date" type="date" required />
          </div>
          <div class="form-group">
            <label>Note (optional)</label>
            <textarea v-model="form.note"></textarea>
          </div>
          <div class="form-actions">
            <button type="submit" :disabled="loading">
              {{ loading ? 'Saving...' : 'Save' }}
            </button>
            <button type="button" @click="closeForm" class="cancel-btn">
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Expense List -->
    <div class="expense-list">
      <h2>Expenses</h2>
      <p v-if="expenses.length === 0">No expenses yet.</p>
      <div v-for="expense in expenses" :key="expense.id" class="expense-item">
        <div class="expense-info">
          <h4>{{ expense.title }}</h4>
          <span class="category">{{ expense.category }}</span>
          <span class="date">{{ expense.date }}</span>
        </div>
        <div class="expense-right">
          <p class="amount">৳{{ expense.amount }}</p>
          <button @click="deleteExpense(expense.id)" class="delete-btn">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import api from '../api/axios'

const router = useRouter()
const expenses = ref([])
const showForm = ref(false)
const loading = ref(false)

const form = ref({
  title: '',
  amount: '',
  category: '',
  date: '',
  note: ''
})

const totalAmount = computed(() =>
  expenses.value.reduce((sum, e) => sum + parseFloat(e.amount), 0)
)

const monthlyAmount = computed(() => {
  const now = new Date()
  const month = now.getMonth()
  const year = now.getFullYear()
  return expenses.value
    .filter(e => {
      const d = new Date(e.date)
      return d.getMonth() === month && d.getFullYear() === year
    })
    .reduce((sum, e) => sum + parseFloat(e.amount), 0)
})

const fetchExpenses = async () => {
  try {
    const response = await api.get('/expenses')
    expenses.value = response.data
  } catch (err) {
    console.error(err)
  }
}

const handleSubmit = async () => {
  loading.value = true
  try {
    await api.post('/expenses', form.value)
    await fetchExpenses()
    closeForm()
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

const deleteExpense = async (id) => {
  if (confirm('Delete this expense?')) {
    await api.delete(`/expenses/${id}`)
    await fetchExpenses()
  }
}

const closeForm = () => {
  showForm.value = false
  form.value = { title: '', amount: '', category: '', date: '', note: '' }
}

const handleLogout = async () => {
  await api.post('/logout')
  localStorage.removeItem('token')
  router.push('/login')
}

onMounted(fetchExpenses)
</script>

<style scoped>
.dashboard { max-width: 800px; margin: 0 auto; padding: 1rem; }
.header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 2rem; }
.logout-btn { background: #ef4444; color: white; border: none; padding: 0.5rem 1rem; border-radius: 4px; cursor: pointer; }
.summary-cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; margin-bottom: 2rem; }
.card { background: white; padding: 1.5rem; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); text-align: center; }
.card .amount { font-size: 1.5rem; font-weight: bold; color: #4f46e5; }
.add-btn { background: #4f46e5; color: white; border: none; padding: 0.75rem 1.5rem; border-radius: 4px; cursor: pointer; margin-bottom: 1rem; }
.modal { position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: flex; justify-content: center; align-items: center; }
.modal-box { background: white; padding: 2rem; border-radius: 8px; width: 100%; max-width: 450px; }
.form-group { margin-bottom: 1rem; }
label { display: block; margin-bottom: 0.5rem; font-weight: 500; }
input, select, textarea { width: 100%; padding: 0.75rem; border: 1px solid #ddd; border-radius: 4px; }
.form-actions { display: flex; gap: 1rem; margin-top: 1rem; }
.form-actions button { flex: 1; padding: 0.75rem; border: none; border-radius: 4px; cursor: pointer; }
.form-actions button[type="submit"] { background: #4f46e5; color: white; }
.cancel-btn { background: #e5e7eb; }
.expense-list { margin-top: 2rem; }
.expense-item { display: flex; justify-content: space-between; align-items: center; background: white; padding: 1rem; border-radius: 8px; margin-bottom: 0.75rem; box-shadow: 0 1px 4px rgba(0,0,0,0.1); }
.category { background: #e0e7ff; color: #4f46e5; padding: 0.25rem 0.5rem; border-radius: 4px; font-size: 0.75rem; margin: 0 0.5rem; }
.date { color: #6b7280; font-size: 0.875rem; }
.expense-right { display: flex; align-items: center; gap: 1rem; }
.expense-right .amount { font-weight: bold; color: #4f46e5; }
.delete-btn { background: #fee2e2; color: #ef4444; border: none; padding: 0.25rem 0.75rem; border-radius: 4px; cursor: pointer; }
</style>