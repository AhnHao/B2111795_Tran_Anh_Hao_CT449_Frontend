<template>
  <div class="container">
    <h2 class="mb-4">Danh sách yêu cầu mượn sách</h2>
    <div class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Tên độc giả</th>
            <th>Tên sách</th>
            <th>Mã sách</th>
            <th>Ngày yêu cầu</th>
            <th>Trạng thái</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="request in requests" :key="request._id">
            <td>{{ request.MaDocGia?.HoTen || 'N/A' }}</td>
            <td>{{ request.TenSach }}</td>
            <td>{{ request.MaSach }}</td>
            <td>{{ formatDate(request.NgayYeuCau) }}</td>
            <td>
              <span class="badge" :class="getStatusBadgeClass(request.TrangThai)">
                {{ request.TrangThai }}
              </span>
            </td>
            <td>
              <button 
                class="btn btn-success btn-sm me-2" 
                @click="approveRequest(request._id)"
                v-if="request.TrangThai === 'chờ duyệt'"
              >
                Duyệt
              </button>
              <button 
                class="btn btn-danger btn-sm"
                @click="rejectRequest(request._id)"
                v-if="request.TrangThai === 'chờ duyệt'"
              >
                Từ chối
              </button>
            </td>
          </tr>
          <tr v-if="requests.length === 0">
            <td colspan="6" class="text-center">Không có yêu cầu mượn sách nào</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import api from '../../services/api';

export default {
  name: 'BorrowRequestList',
  data() {
    return {
      requests: []
    };
  },
  async mounted() {
    await this.loadRequests();
  },
  methods: {
    async loadRequests() {
      try {
        const response = await api.getPendingRequests();
        this.requests = response.data;
      } catch (error) {
        console.error('Lỗi khi tải danh sách yêu cầu:', error);
        alert('Không thể tải danh sách yêu cầu');
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('vi-VN');
    },
    getStatusBadgeClass(status) {
      return {
        'bg-warning': status === 'chờ duyệt',
        'bg-success': status === 'đã duyệt',
        'bg-danger': status === 'từ chối'
      };
    },
    async approveRequest(requestId) {
      try {
        await api.approveRequest(requestId);
        alert('Đã duyệt yêu cầu mượn sách');
        await this.loadRequests();
      } catch (error) {
        console.error('Lỗi khi duyệt yêu cầu:', error);
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra');
      }
    },
    async rejectRequest(requestId) {
      try {
        await api.rejectRequest(requestId);
        alert('Đã từ chối yêu cầu mượn sách');
        await this.loadRequests();
      } catch (error) {
        console.error('Lỗi khi từ chối yêu cầu:', error);
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra');
      }
    }
  }
};
</script>

<style scoped>
.badge {
  padding: 0.5em 1em;
}
</style> 