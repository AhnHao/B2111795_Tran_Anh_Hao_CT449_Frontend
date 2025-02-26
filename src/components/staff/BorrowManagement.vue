<template>
  <div class="container">
    <h2 class="mb-4">Quản lý yêu cầu mượn sách</h2>
    <div class="table-responsive">
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
            <td>{{ request.TenSach }}</td>
            <td>{{ request.MaSach }}</td>
            <td>{{ formatDate(request.NgayYeuCau) }}</td>
            <td>
              <button class="btn btn-success btn-sm me-2" @click="approveRequest(request._id)">
                Duyệt
              </button>
              <button class="btn btn-danger btn-sm" @click="rejectRequest(request._id)">
                Từ chối
              </button>
            </td>
          </tr>
          <tr v-if="pendingRequests.length === 0">
            <td colspan="5" class="text-center">Không có yêu cầu mượn sách nào</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import api from '../../services/api';

export default {
  name: 'BorrowManagement',
  data() {
    return {
      pendingRequests: []
    };
  },
  async mounted() {
    await this.loadPendingRequests();
  },
  methods: {
    async loadPendingRequests() {
      try {
        const response = await api.getPendingRequests();
        this.pendingRequests = response.data;
      } catch (error) {
        console.error('Lỗi khi tải danh sách yêu cầu:', error);
        alert('Không thể tải danh sách yêu cầu');
      }
    },
    formatDate(date) {
      if (!date) return '';
      return new Date(date).toLocaleDateString('vi-VN');
    },
    async approveRequest(requestId) {
      try {
        await api.approveRequest(requestId);
        alert('Đã duyệt yêu cầu mượn sách');
        await this.loadPendingRequests();
      } catch (error) {
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra');
      }
    },
    async rejectRequest(requestId) {
      try {
        await api.rejectRequest(requestId);
        alert('Đã từ chối yêu cầu mượn sách');
        await this.loadPendingRequests();
      } catch (error) {
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra');
      }
    }
  }
};
</script>

<style scoped>
.nav-link {
  cursor: pointer;
}
</style> 