<script setup>
import { ref, computed } from 'vue'

// 1. Mock Data คลังสินค้าภายในออฟฟิศ
const items = ref([
  { id: 'INV001', name: 'MacBook Pro M3', category: 'Electronics', stock: 12, price: 79900 },
  { id: 'INV002', name: 'Dell 27" Monitor', category: 'Electronics', stock: 3, price: 8500 }, // สินค้าใกล้หมด
  { id: 'INV003', name: 'Ergonomic Office Chair', category: 'Furniture', stock: 25, price: 4500 },
  { id: 'INV004', name: 'Logitech MX Master 3S', category: 'Accessories', stock: 0, price: 3900 }, // สินค้าหมดสต็อก
  { id: 'INV005', name: 'Mechanical Keyboard', category: 'Accessories', stock: 15, price: 2900 }
])

// 2. State สำหรับฟอร์มเพิ่มสินค้าและกล่องค้นหา
const search = ref('')
const newItem = ref({
  name: '',
  category: 'Electronics',
  stock: 1,
  price: 0
})

// 3. Computed Properties คำนวณตัวเลขแดชบอร์ดอัตโนมัติ (จุดขายความโปร!)
const totalItems = computed(() => items.value.length)
const lowStockCount = computed(() => items.value.filter(i => i.stock <= 5).length)
const totalValue = computed(() => {
  return items.value.reduce((sum, item) => sum + (item.price * item.stock), 0)
})

// ฟังก์ชันกรองข้อมูลตามคำค้นหา
const filteredItems = computed(() => {
  return items.value.filter(item => 
    item.name.toLowerCase().includes(search.value.toLowerCase()) ||
    item.category.toLowerCase().includes(search.value.toLowerCase())
  )
})

// 4. ฟังก์ชันจัดการข้อมูล (CRUD Logic)
const addItem = () => {
  if (!newItem.value.name || newItem.value.price <= 0) return
  
  const nextId = 'INV' + String(items.value.length + 1).padStart(3, '0')
  items.value.push({
    id: nextId,
    ...newItem.value
  })

  // รีเซ็ตฟอร์มหลังเพิ่มเสร็จ
  newItem.value = { name: '', category: 'Electronics', stock: 1, price: 0 }
}

const deleteItem = (id) => {
  items.value = items.value.filter(item => item.id !== id)
}

const adjustStock = (id, amount) => {
  const item = items.value.find(i => i.id === id)
  if (item) {
    item.stock = Math.max(0, item.stock + amount) // ป้องกันไม่ให้สต็อกติดลบ
  }
}
</script>

<template>
  <div class="min-h-screen bg-slate-50 text-slate-800 font-sans antialiased">
    <!-- Navbar -->
    <nav class="bg-indigo-950 text-white p-4 shadow-md">
      <div class="max-w-7xl mx-auto flex justify-between items-center">
        <h1 class="text-xl font-bold tracking-wider flex items-center gap-2">
          <span>📦</span> Smart Office Inventory
        </h1>
        <span class="text-xs bg-indigo-800 px-3 py-1 rounded-full text-indigo-200">Vue 3 Composition API</span>
      </div>
    </nav>

    <div class="max-w-7xl mx-auto p-4 md:p-6 space-y-6">
      
      <!-- 📊 ส่วนแดชบอร์ดการคำนวณข้อมูล (Metrics Cards) -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
          <p class="text-sm font-medium text-slate-500">รายการอุปกรณ์ทั้งหมด</p>
          <p class="text-3xl font-bold text-slate-900 mt-1">{{ totalItems }} ชิ้น</p>
        </div>
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
          <p class="text-sm font-medium text-slate-500">อุปกรณ์ที่สต็อกใกล้หมด (≤ 5)</p>
          <p class="text-3xl font-bold text-rose-600 mt-1">{{ lowStockCount }} รายการ</p>
        </div>
        <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
          <p class="text-sm font-medium text-slate-500">มูลค่าคลังสินค้ารวม</p>
          <p class="text-3xl font-bold text-emerald-600 mt-1">฿{{ totalValue.toLocaleString() }}</p>
        </div>
      </div>

      <!-- ส่วนควบคุมและฟอร์มทำงาน -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <!-- ฝั่งซ้าย: ฟอร์มเพิ่มสินค้าใหม่ (Create) -->
        <div class="lg:col-span-1 bg-white p-5 rounded-2xl shadow-sm border border-slate-200 h-fit">
          <h2 class="text-base font-bold text-slate-900 mb-4">➕ เพิ่มอุปกรณ์เข้าคลัง</h2>
          <form @submit.prevent="addItem" class="space-y-4">
            <div>
              <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่ออุปกรณ์</label>
              <input v-model="newItem.name" type="text" placeholder="เช่น Logitech Mouse" class="w-full p-2.5 border border-slate-200 rounded-xl text-sm focus:outline-indigo-600" required />
            </div>
            <div>
              <label class="block text-xs font-semibold text-slate-600 mb-1">หมวดหมู่</label>
              <select v-model="newItem.category" class="w-full p-2.5 border border-slate-200 rounded-xl text-sm focus:outline-indigo-600">
                <option value="Electronics">Electronics</option>
                <option value="Furniture">Furniture</option>
                <option value="Accessories">Accessories</option>
              </select>
            </div>
            <div class="grid grid-cols-2 gap-2">
              <div>
                <label class="block text-xs font-semibold text-slate-600 mb-1">จำนวนเริ่มต้น</label>
                <input v-model.number="newItem.stock" type="number" min="1" class="w-full p-2.5 border border-slate-200 rounded-xl text-sm focus:outline-indigo-600" />
              </div>
              <div>
                <label class="block text-xs font-semibold text-slate-600 mb-1">ราคาต่อชิ้น (฿)</label>
                <input v-model.number="newItem.price" type="number" min="0" class="w-full p-2.5 border border-slate-200 rounded-xl text-sm focus:outline-indigo-600" required />
              </div>
            </div>
            <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-sm py-2.5 rounded-xl transition-colors mt-2 shadow-sm">
              บันทึกข้อมูลลงระบบ
            </button>
          </form>
        </div>

        <!-- ฝั่งขวา: ตารางแสดงรายการและการค้นหา (Read, Update, Delete) -->
        <div class="lg:col-span-2 bg-white p-5 rounded-2xl shadow-sm border border-slate-200 space-y-4">
          <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-2 border-b border-slate-100 pb-3">
            <h2 class="text-base font-bold text-slate-900">📋 รายการสิ่งของในคลัง</h2>
            <input v-model="search" type="text" placeholder="🔍 ค้นหาชื่อหรือหมวดหมู่..." class="p-2 border border-slate-200 rounded-xl text-xs w-full sm:w-64 focus:outline-indigo-600" />
          </div>

          <!-- Table Container -->
          <div class="overflow-x-auto">
            <table class="w-full text-left border-collapse text-sm">
              <thead>
                <tr class="text-slate-400 text-xs uppercase bg-slate-50 border-b border-slate-100">
                  <th class="p-3">รหัส</th>
                  <th class="p-3">ชื่อสินค้า</th>
                  <th class="p-3">หมวดหมู่</th>
                  <th class="p-3">จำนวนสต็อก</th>
                  <th class="p-3">ราคา</th>
                  <th class="p-3 text-right">จัดการ</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr v-for="item in filteredItems" :key="item.id" class="hover:bg-slate-50/80 transition-colors">
                  <td class="p-3 font-mono text-xs text-slate-500">{{ item.id }}</td>
                  <td class="p-3 font-semibold text-slate-900">{{ item.name }}</td>
                  <td class="p-3 text-xs"><span class="bg-slate-100 text-slate-600 px-2 py-1 rounded-md">{{ item.category }}</span></td>
                  <td class="p-3">
                    <div class="flex items-center space-x-2">
                      <!-- ป้ายสถานะแจ้งเตือนสต็อก -->
                      <span :class="[
                        'px-2 py-0.5 rounded-full text-xs font-medium',
                        item.stock === 0 ? 'bg-rose-100 text-rose-700' :
                        item.stock <= 5 ? 'bg-amber-100 text-amber-700' : 'bg-emerald-100 text-emerald-700'
                      ]">
                        {{ item.stock }} ชิ้น
                      </span>
                      <!-- ปุ่มปรับจำนวนแบบด่วน -->
                      <button @click="adjustStock(item.id, -1)" class="w-5 h-5 bg-slate-100 rounded hover:bg-slate-200 flex items-center justify-center font-bold text-xs">-</button>
                      <button @click="adjustStock(item.id, 1)" class="w-5 h-5 bg-slate-100 rounded hover:bg-slate-200 flex items-center justify-center font-bold text-xs">+</button>
                    </div>
                  </td>
                  <td class="p-3 font-medium text-slate-600">฿{{ item.price.toLocaleString() }}</td>
                  <td class="p-3 text-right">
                    <button @click="deleteItem(item.id)" class="text-xs font-semibold text-rose-600 hover:text-rose-800 transition-colors">
                      🗑️ ลบ
                    </button>
                  </td>
                </tr>
                <tr v-if="filteredItems.length === 0">
                  <td colspan="6" class="p-8 text-center text-slate-400">ไม่พบข้อมูลอุปกรณ์ที่ค้นหา</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>