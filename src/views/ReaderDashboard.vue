<template>
  <div class="container-fluid" style="padding: 0">
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary mb-4">
      <div class="container">
        <a class="navbar-brand">Thư viện</a>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav me-auto">
            <li class="nav-item">
              <a
                class="nav-link"
                :class="{ active: activeTab === 'books' }"
                @click="activeTab = 'books'"
              >
                Danh sách sách
              </a>
            </li>
            <li class="nav-item">
              <a
                class="nav-link"
                :class="{ active: activeTab === 'history' }"
                @click="activeTab = 'history'"
              >
                Quản lý mượn sách
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
      <BookList
        v-if="activeTab === 'books'"
        @borrow-requested="activeTab = 'history'"
      />
      <BorrowingHistory v-if="activeTab === 'history'" />
    </div>
  </div>
</template>

<script>
import BookList from "../components/reader/BookList.vue";
import BorrowingHistory from "../components/reader/BorrowingHistory.vue";

export default {
  name: "ReaderDashboard",
  components: {
    BookList,
    BorrowingHistory,
  },
  data() {
    return {
      activeTab: "books",
    };
  },
  methods: {
    handleLogout() {
      localStorage.removeItem("token");
      localStorage.removeItem("userRole");
      localStorage.removeItem("userData");
      this.$router.push("/login");
    },
  },
};
</script>

<style scoped>
.nav-link {
  cursor: pointer;
}
</style>
