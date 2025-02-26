<template>
  <div class="container-fluid">
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary mb-4">
      <div class="container">
        <a class="navbar-brand">Quản lý thư viện</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav me-auto">
            <li class="nav-item">
              <a class="nav-link" :class="{ active: activeTab === 'books' }" @click="activeTab = 'books'">
                Quản lý sách
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link" :class="{ active: activeTab === 'publishers' }" @click="activeTab = 'publishers'">
                Quản lý NXB
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link" :class="{ active: activeTab === 'borrowings' }" @click="activeTab = 'borrowings'">
                Quản lý mượn trả
              </a>
            </li>
          </ul>
          <ul class="navbar-nav">
            <li class="nav-item">
              <a class="nav-link" @click="handleLogout">Đăng xuất</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <div class="container">
      <BookManagement v-if="activeTab === 'books'" />
      <PublisherManagement v-if="activeTab === 'publishers'" />
      <BorrowManagement v-if="activeTab === 'borrowings'" />
    </div>
  </div>
</template>

<script>
import BookManagement from '../components/staff/BookManagement.vue';
import PublisherManagement from '../components/staff/PublisherManagement.vue';
import BorrowManagement from '../components/staff/BorrowManagement.vue';

export default {
  name: 'StaffDashboard',
  components: {
    BookManagement,
    PublisherManagement,
    BorrowManagement
  },
  data() {
    return {
      activeTab: 'books'
    }
  },
  methods: {
    handleLogout() {
      localStorage.removeItem('token');
      localStorage.removeItem('userRole');
      localStorage.removeItem('userData');
      this.$router.push('/login');
    }
  }
}
</script>

<style scoped>
.nav-link {
  cursor: pointer;
}
</style>
