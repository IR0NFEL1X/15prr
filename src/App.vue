<template>
  <div>
    <nav class="navbar navbar-dark bg-espresso sticky-top shadow-sm">
      <div class="container">
        <a class="navbar-brand d-flex align-items-center" href="#">
          <span class="badge bg-warning text-dark rounded-circle me-2">B</span>
          Brew & Soul
        </a>
        <button @click="openModal(null)" class="btn btn-coffee">
          + Добавить кофе
        </button>
      </div>
    </nav>

    <header class="bg-espresso text-white py-5 mb-4 text-center">
      <div class="container">
        <h1 class="display-4 fw-bold">Каталог обжарки</h1>
        <p class="lead text-crema">Ваша личная коллекция кофейных впечатлений</p>
        <div class="d-flex justify-center gap-4 mt-3 justify-content-center">
          <div><strong>{{ coffees.length }}</strong> <small class="text-uppercase d-block">Сортов</small></div>
          <div class="vr"></div>
          <div><strong>{{ avgRating }}</strong> <small class="text-uppercase d-block">Ср. оценка</small></div>
        </div>
      </div>
    </header>

    <div class="container mb-5">
      <div class="row g-3 bg-white p-3 rounded shadow-sm">
        <div class="col-md-6">
          <input v-model="searchQuery" type="text" class="form-control" placeholder="🔍 Поиск по названию или описанию...">
        </div>
        <div class="col-md-6">
          <div class="input-group">
            <label class="input-group-text">Сортировка</label>
            <select v-model="sortBy" class="form-select">
              <option value="newest">Сначала новые</option>
              <option value="rating-desc">По оценке ↓</option>
              <option value="name-az">По названию (А-Я)</option>
            </select>
          </div>
        </div>
      </div>
    </div>

    <main class="container mb-5">
      <div v-if="filteredCoffees.length === 0" class="text-center py-5">
        <p class="text-muted">Ничего не найдено...</p>
      </div>

      <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
        <div v-for="coffee in filteredCoffees" :key="coffee.id" class="col">
          <div class="card h-100 shadow-sm coffee-card border-0">
            <img v-if="coffee.photo" :src="coffee.photo" class="card-img-top" :alt="coffee.name">
            <div v-else class="card-img-top bg-light d-flex align-items-center justify-content-center" style="height:200px">
              <span class="display-1">☕</span>
            </div>
            
            <div class="card-body">
              <div class="d-flex justify-content-between align-items-start mb-2">
                <h5 class="card-title mb-0 fw-bold">{{ coffee.name }}</h5>
                <span class="badge bg-warning text-dark">★ {{ coffee.rating }}</span>
              </div>
              <p class="card-text text-muted small">{{ coffee.description }}</p>
            </div>
            
            <div class="card-footer bg-white border-top-0 d-flex justify-content-between pb-3">
              <button @click="openModal(coffee)" class="btn btn-outline-secondary btn-sm">Редактировать</button>
              <button @click="deleteCoffee(coffee.id)" class="btn btn-outline-danger btn-sm">Удалить</button>
            </div>
          </div>
        </div>
      </div>
    </main>

    <div v-if="showModal" class="custom-modal-overlay" @click.self="closeModal">
      <div class="modal-dialog w-100" style="max-width: 500px;">
        <div class="modal-content shadow-lg border-0">
          <div class="modal-header bg-espresso text-white">
            <h5 class="modal-title">{{ editingCoffee ? 'Изменить сорт' : 'Новый кофе' }}</h5>
            <button type="button" class="btn-close btn-close-white" @click="closeModal"></button>
          </div>
          <div class="modal-body p-4">
            <form @submit.prevent="saveCoffee">
              <div class="mb-3">
                <label class="form-label small fw-bold">Название *</label>
                <input v-model="form.name" type="text" class="form-control" required placeholder="Напр: Эфиопия">
              </div>
              <div class="mb-3">
                <label class="form-label small fw-bold">Описание</label>
                <textarea v-model="form.description" class="form-control" rows="3"></textarea>
              </div>
              <div class="row">
                <div class="col-6 mb-3">
                  <label class="form-label small fw-bold">Оценка (1-5)</label>
                  <input v-model.number="form.rating" type="number" min="1" max="5" class="form-control">
                </div>
                <div class="col-6 mb-3">
                  <label class="form-label small fw-bold">URL Фото</label>
                  <input v-model="form.photo" type="url" class="form-control" placeholder="https://...">
                </div>
              </div>
              <div class="d-grid gap-2 mt-4">
                <button type="submit" class="btn btn-coffee">Сохранить в коллекцию</button>
                <button type="button" @click="closeModal" class="btn btn-light">Отмена</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive, onMounted } from 'vue';

const STORAGE_KEY = 'brew_soul_bootstrap_v1';
const coffees = ref([]);
const showModal = ref(false);
const editingCoffee = ref(null);
const searchQuery = ref('');
const sortBy = ref('newest');

const form = reactive({
  name: '', rating: 5, description: '', photo: ''
});

onMounted(() => {
  const data = localStorage.getItem(STORAGE_KEY);
  if (data) {
    coffees.value = JSON.parse(data);
  } else {
    // Тестовые данные
    coffees.value = [
      { id: 1, name: 'Колумбия Уила', rating: 5, description: 'Сбалансированный вкус с нотами шоколада.', photo: 'https://images.unsplash.com/photo-1559056199-641a0ac8b55e?w=500', createdAt: Date.now() }
    ];
  }
});

const save = () => localStorage.setItem(STORAGE_KEY, JSON.stringify(coffees.value));

const avgRating = computed(() => {
  if (!coffees.value.length) return 0;
  return (coffees.value.reduce((a, b) => a + b.rating, 0) / coffees.value.length).toFixed(1);
});

const filteredCoffees = computed(() => {
  let list = [...coffees.value];
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase();
    list = list.filter(c => c.name.toLowerCase().includes(q) || c.description.toLowerCase().includes(q));
  }
  list.sort((a, b) => {
    if (sortBy.value === 'rating-desc') return b.rating - a.rating;
    if (sortBy.value === 'name-az') return a.name.localeCompare(b.name);
    return b.createdAt - a.createdAt;
  });
  return list;
});

const openModal = (coffee) => {
  if (coffee) {
    editingCoffee.value = coffee;
    Object.assign(form, coffee);
  } else {
    editingCoffee.value = null;
    Object.assign(form, { name: '', rating: 5, description: '', photo: '' });
  }
  showModal.value = true;
};

const closeModal = () => showModal.value = false;

const saveCoffee = () => {
  if (editingCoffee.value) {
    const idx = coffees.value.findIndex(c => c.id === editingCoffee.value.id);
    coffees.value[idx] = { ...editingCoffee.value, ...form };
  } else {
    coffees.value.unshift({ id: Date.now(), ...form, createdAt: Date.now() });
  }
  save();
  closeModal();
};

const deleteCoffee = (id) => {
  if (confirm('Удалить этот сорт?')) {
    coffees.value = coffees.value.filter(c => c.id !== id);
    save();
  }
};
</script>