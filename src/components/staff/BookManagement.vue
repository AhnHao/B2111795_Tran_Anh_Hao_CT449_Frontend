<template>
  <div class="container">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h2>Quản lý sách</h2>
      <button class="btn btn-primary" @click="showAddModal">
        <i class="bi bi-plus-lg"></i> Thêm sách
      </button>
    </div>

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
          />
          <button class="btn btn-outline-secondary" type="button">
            <i class="bi bi-search"></i>
          </button>
        </div>
      </div>
    </div>

    <!-- Bảng danh sách -->
    <div class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Mã sách</th>
            <th>Tên sách</th>
            <th>Tác giả</th>
            <th>Nhà xuất bản</th>
            <th>Đơn giá</th>
            <th>Số quyển</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="book in books" :key="book._id">
            <td>{{ book.MaSach }}</td>
            <td>{{ book.TenSach }}</td>
            <td>{{ book.TacGia }}</td>
            <td>{{ book.MaNXB?.TenNXB || "Không xác định" }}</td>
            <td>{{ formatCurrency(book.DonGia) }}</td>
            <td>{{ book.SoQuyen }}</td>
            <td>
              <button
                class="btn btn-sm btn-warning me-2"
                @click="showEditModal(book)"
              >
                <i class="bi bi-pencil"></i>
              </button>
              <button
                class="btn btn-sm btn-danger"
                @click="confirmDelete(book)"
              >
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal thêm/sửa -->
    <div class="modal fade" id="bookModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              {{ isEditing ? "Sửa sách" : "Thêm sách" }}
            </h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="handleSubmit">
              <div class="mb-3">
                <label class="form-label">Tên sách</label>
                <input
                  type="text"
                  class="form-control"
                  v-model="formData.TenSach"
                  required
                />
              </div>
              <div class="mb-3">
                <label class="form-label">Tác giả</label>
                <input
                  type="text"
                  class="form-control"
                  v-model="formData.TacGia"
                  required
                />
              </div>
              <div class="mb-3">
                <label class="form-label">Nhà xuất bản</label>
                <select class="form-select" v-model="formData.MaNXB" required>
                  <option value="">Chọn nhà xuất bản</option>
                  <option
                    v-for="pub in publishers"
                    :key="pub._id"
                    :value="pub.MaNXB"
                  >
                    {{ pub.TenNXB }}
                  </option>
                </select>
              </div>
              <div class="mb-3">
                <label class="form-label">Đơn giá</label>
                <input
                  type="number"
                  class="form-control"
                  v-model="formData.DonGia"
                  required
                />
              </div>
              <div class="mb-3">
                <label class="form-label">Số quyển</label>
                <input
                  type="number"
                  class="form-control"
                  v-model="formData.SoQuyen"
                  required
                />
              </div>
              <div class="alert alert-danger" v-if="error">{{ error }}</div>
              <div class="text-end">
                <button
                  type="button"
                  class="btn btn-secondary me-2"
                  data-bs-dismiss="modal"
                >
                  Hủy
                </button>
                <button
                  type="submit"
                  class="btn btn-primary"
                  :disabled="loading"
                >
                  {{ loading ? "Đang xử lý..." : "Lưu" }}
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal xác nhận xóa -->
    <div class="modal fade" id="deleteModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Xác nhận xóa</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
            ></button>
          </div>
          <div class="modal-body">Bạn có chắc chắn muốn xóa sách này?</div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Hủy
            </button>
            <button
              type="button"
              class="btn btn-danger"
              @click="handleDelete"
              :disabled="loading"
            >
              {{ loading ? "Đang xử lý..." : "Xóa" }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Modal } from "bootstrap";
import api from "../../services/api";

export default {
  name: "BookManagement",
  data() {
    return {
      books: [],
      publishers: [],
      formData: {
        TenSach: "",
        TacGia: "",
        MaNXB: "",
        DonGia: "",
        SoQuyen: "",
      },
      selectedBook: null,
      isEditing: false,
      loading: false,
      error: null,
      bookModal: null,
      deleteModal: null,
      searchKeyword: "",
    };
  },
  async mounted() {
    await Promise.all([this.loadBooks(), this.loadPublishers()]);
    this.bookModal = new Modal(document.getElementById("bookModal"));
    this.deleteModal = new Modal(document.getElementById("deleteModal"));
  },
  methods: {
    async loadBooks() {
      try {
        const response = await api.getAllBooks();
        console.log("Books loaded:", response.data);
        this.books = response.data;
      } catch (error) {
        console.error("Lỗi khi tải danh sách sách:", error);
      }
    },
    async loadPublishers() {
      try {
        const response = await api.getAllPublishers();
        console.log("Publishers loaded:", response.data);
        this.publishers = response.data;
      } catch (error) {
        console.error("Lỗi khi tải danh sách nhà xuất bản:", error);
      }
    },
    showAddModal() {
      this.isEditing = false;
      this.formData = {
        TenSach: "",
        TacGia: "",
        MaNXB: "",
        DonGia: "",
        SoQuyen: "",
      };
      this.error = null;
      this.bookModal.show();
    },
    showEditModal(book) {
      this.isEditing = true;
      this.selectedBook = book;
      this.formData = {
        TenSach: book.TenSach,
        TacGia: book.TacGia,
        MaNXB: book.MaNXB?._id || "",
        DonGia: book.DonGia,
        SoQuyen: book.SoQuyen,
      };
      this.error = null;
      this.bookModal.show();
    },
    async handleSubmit() {
      this.loading = true;
      this.error = null;
      try {
        const bookData = {
          ...this.formData,
          DonGia: Number(this.formData.DonGia),
          SoQuyen: Number(this.formData.SoQuyen),
        };
        console.log("Submitting book data:", bookData);

        if (this.isEditing) {
          await api.updateBook(this.selectedBook._id, bookData);
        } else {
          await api.createBook(bookData);
        }
        await this.loadBooks();
        this.bookModal.hide();
      } catch (error) {
        console.error("Error submitting book:", error);
        this.error = error.response?.data?.message || "Đã có lỗi xảy ra";
      } finally {
        this.loading = false;
      }
    },
    confirmDelete(book) {
      this.selectedBook = book;
      this.deleteModal.show();
    },
    async handleDelete() {
      this.loading = true;
      try {
        await api.deleteBook(this.selectedBook._id);
        await this.loadBooks();
        this.deleteModal.hide();
      } catch (error) {
        alert(error.response?.data?.message || "Đã có lỗi xảy ra");
      } finally {
        this.loading = false;
      }
    },
    async handleSearch() {
      if (this.searchKeyword.trim()) {
        try {
          const response = await api.searchBooks(this.searchKeyword);
          this.books = response.data;
        } catch (error) {
          console.error("Lỗi khi tìm kiếm sách:", error);
        }
      } else {
        await this.loadBooks();
      }
    },
    formatCurrency(value) {
      return new Intl.NumberFormat("vi-VN", {
        style: "currency",
        currency: "VND",
      }).format(value);
    },
  },
};
</script>
