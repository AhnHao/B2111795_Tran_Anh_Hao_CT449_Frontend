<template>
  <div class="container">
    <h2 class="mb-4">Quản lý yêu cầu mượn sách</h2>

    <!-- Tab Navigation -->
    <ul class="nav nav-tabs mb-4">
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'pending' }"
          @click="switchTab('pending')"
          href="#"
        >
          Duyệt yêu cầu mượn sách
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
    </ul>

    <!-- Pending Requests Table -->
    <div v-if="activeTab === 'pending'" class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th>Độc giả</th>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày yêu cầu</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="request in pendingRequests" :key="request._id">
            <td>{{ request.MaDocGia?.HoTen }}</td>
            <td>{{ request.MaSach?.TenSach }}</td>
            <td>{{ request.MaSach?.MaSach }}</td>
            <td>{{ formatDate(request.NgayYeuCau) }}</td>
            <td>
              <button
                class="btn btn-success btn-sm me-2"
                @click="approveRequest(request._id)"
              >
                Duyệt
              </button>
              <button
                class="btn btn-danger btn-sm"
                @click="rejectRequest(request._id)"
              >
                Từ chối
              </button>
            </td>
          </tr>
          <tr v-if="pendingRequests.length === 0">
            <td colspan="5" class="text-center">
              Không có yêu cầu mượn sách nào
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Borrowing History Table -->
    <div v-if="activeTab === 'history'" class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Độc giả</th>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày mượn</th>
            <th>Ngày trả</th>
            <th>Trạng thái</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="history in borrowingHistory" :key="history._id">
            <td>{{ history.MaDocGia?.HoTen }}</td>
            <td>{{ history.MaSach?.TenSach }}</td>
            <td>{{ history.MaSach?.MaSach }}</td>
            <td>{{ formatDate(history.NgayMuon) }}</td>
            <td>{{ formatDate(history.NgayTra) }}</td>
            <td>
              <span
                class="badge"
                :class="getStatusBadgeClass(history.TrangThai)"
              >
                {{ history.TrangThai }}
              </span>
            </td>
          </tr>
          <tr v-if="borrowingHistory.length === 0">
            <td colspan="6" class="text-center">Không có lịch sử mượn sách</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import api from "../../services/api";

export default {
  name: "BorrowManagement",
  data() {
    return {
      activeTab: "pending",
      pendingRequests: [],
      borrowingHistory: [],
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
        if (this.activeTab === "pending") {
          const response = await api.getPendingRequests();
          this.pendingRequests = response.data;
        } else if (this.activeTab === "history") {
          const response = await api.getAllBorrowingHistory();
          this.borrowingHistory = response.data;
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
    async approveRequest(requestId) {
      try {
        await api.approveRequest(requestId);
        alert("Đã duyệt yêu cầu mượn sách");
        await this.loadData();
      } catch (error) {
        alert(error.response?.data?.message || "Đã có lỗi xảy ra");
      }
    },
    async rejectRequest(requestId) {
      try {
        await api.rejectRequest(requestId);
        alert("Đã từ chối yêu cầu mượn sách");
        await this.loadData();
      } catch (error) {
        alert(error.response?.data?.message || "Đã có lỗi xảy ra");
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
