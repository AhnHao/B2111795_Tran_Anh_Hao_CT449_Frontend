<template>
  <div class="container">
    <!-- Thanh tìm kiếm -->
    <div class="row mb-4">
      <div class="col-md-6">
        <div class="input-group">
          <input 
            type="text" 
            class="form-control" 
            placeholder="Tìm kiếm sách..." 
            v-model="searchKeyword"
            @input="handleSearch"
          >
        </div>
      </div>
    </div>

    <!-- Danh sách sách -->
    <div class="row row-cols-1 row-cols-md-3 g-4">
      <div v-for="book in books" :key="book._id" class="col">
        <div class="card h-100">
          <div class="card-body">
            <h5 class="card-title">{{ book.TenSach }}</h5>
            <p class="card-text">
              <small class="text-muted">Mã sách: {{ book.MaSach }}</small>
            </p>
            <p class="card-text">
              Nhà xuất bản: {{ book.MaNXB?.TenNXB || 'Không xác định' }}
            </p>
            <p class="card-text">Đơn giá: {{ book.DonGia }}đ</p>
            <p class="card-text">Số quyển: {{ book.SoQuyen }}</p>
            <p class="card-text" v-if="borrowStatus[book.MaSach]">
              <span class="badge" :class="getStatusBadgeClass(book.MaSach)">
                {{ getStatusText(book.MaSach) }}
              </span>
            </p>
          </div>
          <div class="card-footer">
            <button 
              class="btn btn-primary w-100" 
              @click="requestBorrow(book.MaSach)"
              :disabled="borrowStatus[book.MaSach] === 'unavailable' || borrowStatus[book.MaSach] === 'pending'"
            >
              {{ getButtonText(book.MaSach) }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../../services/api';

export default {
  name: 'BookList',
  data() {
    return {
      books: [],
      searchKeyword: '',
      borrowStatus: {}
    };
  },
  async mounted() {
    await this.loadBooks();
    await this.loadBookStatus();
  },
  methods: {
    async loadBooks() {
      try {
        const response = await api.getAllBooks();
        this.books = response.data;
      } catch (error) {
        console.error('Lỗi khi tải danh sách sách:', error);
        alert('Không thể tải danh sách sách');
      }
    },
    async loadBookStatus() {
      try {
        const [borrowing, pending] = await Promise.all([
          api.getBorrowingBooks(),
          api.getMyPendingRequests()
        ]);

        const status = {};
        borrowing.data.forEach(book => {
          status[book.MaSach] = 'đang mượn';
        });
        pending.data.forEach(book => {
          status[book.MaSach] = 'chờ duyệt';
        });

        this.borrowStatus = status;
      } catch (error) {
        console.error('Lỗi khi tải trạng thái sách:', error);
      }
    },
    getStatusBadgeClass(maSach) {
      const status = this.borrowStatus[maSach];
      return {
        'bg-warning': status === 'pending',
        'bg-danger': status === 'unavailable',
        'bg-success': !status
      };
    },
    getStatusText(maSach) {
      const status = this.borrowStatus[maSach];
      switch (status) {
        case 'unavailable':
          return 'Đã có người mượn';
        case 'pending':
          return 'Đang chờ duyệt';
        default:
          return 'Có thể mượn';
      }
    },
    getButtonText(maSach) {
      const status = this.borrowStatus[maSach];
      switch (status) {
        case 'unavailable':
          return 'Đã có người mượn';
        case 'pending':
          return 'Đang chờ duyệt';
        default:
          return 'Yêu cầu mượn sách';
      }
    },
    async requestBorrow(maSach) {
      try {
        const response = await api.requestBorrow(maSach);
        alert('Đã gửi yêu cầu mượn sách thành công');
        await this.loadBookStatus();
      } catch (error) {
        console.error('Lỗi khi gửi yêu cầu mượn:', error);
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra khi gửi yêu cầu mượn');
      }
    },
    async handleSearch() {
      if (this.searchKeyword.trim()) {
        try {
          const response = await api.searchBooks(this.searchKeyword);
          this.books = response.data;
        } catch (error) {
          console.error('Lỗi khi tìm kiếm sách:', error);
        }
      } else {
        await this.loadBooks();
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
