<template>
  <div class="container">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h2>Lịch sử mượn sách</h2>
    </div>

    <!-- Tab Navigation -->
    <ul class="nav nav-tabs mb-4">
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'borrowing' }"
          @click="switchTab('borrowing')"
          href="#"
        >
          Đang mượn
        </a>
      </li>
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'history' }"
          @click="switchTab('history')"
          href="#"
        >
          Lịch sử mượn
        </a>
      </li>
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'pending' }"
          @click="switchTab('pending')"
          href="#"
        >
          Yêu cầu chờ duyệt
        </a>
      </li>
    </ul>

    <!-- Borrowing Books Table -->
    <div v-if="activeTab === 'borrowing'" class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày mượn</th>
            <th>Ngày hẹn trả</th>
            <th>Trạng thái</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="book in borrowingBooks" :key="book._id">
            <td>{{ book.MaSach?.TenSach }}</td>
            <td>{{ book.MaSach?.MaSach }}</td>
            <td>{{ formatDate(book.NgayMuon) }}</td>
            <td>{{ formatDate(book.NgayHenTra) }}</td>
            <td>
              <span class="badge bg-primary">Đang mượn</span>
            </td>
            <td>
              <button
                class="btn btn-success btn-sm"
                @click="returnBook(book._id)"
              >
                Trả sách
              </button>
            </td>
          </tr>
          <tr v-if="borrowingBooks.length === 0">
            <td colspan="6" class="text-center">Không có sách đang mượn</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- History Table -->
    <div v-if="activeTab === 'history'" class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày mượn</th>
            <th>Ngày trả</th>
            <th>Trạng thái</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="book in historyBooks" :key="book._id">
            <td>{{ book.MaSach?.TenSach }}</td>
            <td>{{ book.MaSach?.MaSach }}</td>
            <td>{{ formatDate(book.NgayMuon) }}</td>
            <td>{{ formatDate(book.NgayTra) }}</td>
            <td>
              <span class="badge" :class="getStatusBadgeClass(book.TrangThai)">
                {{ book.TrangThai }}
              </span>
            </td>
          </tr>
          <tr v-if="historyBooks.length === 0">
            <td colspan="5" class="text-center">Không có lịch sử mượn sách</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pending Requests Table -->
    <div v-if="activeTab === 'pending'" class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày yêu cầu</th>
            <th>Trạng thái</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="request in pendingRequests" :key="request._id">
            <td>{{ request.MaSach?.TenSach }}</td>
            <td>{{ request.MaSach?.MaSach }}</td>
            <td>{{ formatDate(request.NgayYeuCau) }}</td>
            <td>
              <span class="badge bg-warning">Chờ duyệt</span>
            </td>
          </tr>
          <tr v-if="pendingRequests.length === 0">
            <td colspan="4" class="text-center">Không có yêu cầu chờ duyệt</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import api from "../../services/api";

export default {
  name: "BorrowingList",
  data() {
    return {
      activeTab: "borrowing",
      borrowingBooks: [],
      historyBooks: [],
      pendingRequests: [],
    };
  },
  async mounted() {
    await this.loadData();
  },
  methods: {
    async switchTab(tab) {
      this.activeTab = tab;
      await this.loadData();
    },
    async loadData() {
      try {
        switch (this.activeTab) {
          case "borrowing":
            const borrowingResponse = await api.getMyBorrowingBooks();
            this.borrowingBooks = borrowingResponse.data;
            break;
          case "history":
            const historyResponse = await api.getMyBorrowingHistory();
            this.historyBooks = historyResponse.data;
            break;
          case "pending":
            const pendingResponse = await api.getMyPendingRequests();
            this.pendingRequests = pendingResponse.data;
            break;
        }
      } catch (error) {
        console.error("Lỗi khi tải dữ liệu:", error);
        alert("Không thể tải dữ liệu");
      }
    },
    formatDate(date) {
      if (!date) return "";
      return new Date(date).toLocaleDateString("vi-VN");
    },
    getStatusBadgeClass(status) {
      return {
        "bg-success": status === "đã trả",
        "bg-danger": status === "từ chối",
        "bg-warning": status === "chờ duyệt",
        "bg-primary": status === "đã duyệt",
      };
    },
    async returnBook(requestId) {
      try {
        await api.returnBook(requestId);
        alert("Đã trả sách thành công");
        await this.loadData();
      } catch (error) {
        console.error("Lỗi khi trả sách:", error);
        alert(error.response?.data?.message || "Không thể trả sách");
      }
    },
  },
};
</script>

<style scoped>
.nav-link {
  cursor: pointer;
}
.badge {
  padding: 0.5em 1em;
}
</style>
