<template>
  <div class="container">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h2>Quản lý nhà xuất bản</h2>
      <button class="btn btn-primary" @click="showAddModal">
        <i class="bi bi-plus-lg"></i> Thêm nhà xuất bản
      </button>
    </div>

    <!-- Bảng danh sách -->
    <div class="table-responsive">
      <table class="table table-striped">
        <thead>
          <tr>
            <th>Mã NXB</th>
            <th>Tên nhà xuất bản</th>
            <th>Địa chỉ</th>
            <th>Thao tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="publisher in publishers" :key="publisher._id">
            <td>{{ publisher.MaNXB }}</td>
            <td>{{ publisher.TenNXB }}</td>
            <td>{{ publisher.DiaChi }}</td>
            <td>
              <button class="btn btn-sm btn-warning me-2" @click="showEditModal(publisher)">
                <i class="bi bi-pencil"></i>
              </button>
              <button class="btn btn-sm btn-danger" @click="confirmDelete(publisher)">
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal thêm/sửa -->
    <div class="modal fade" id="publisherModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ isEditing ? 'Sửa nhà xuất bản' : 'Thêm nhà xuất bản' }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="handleSubmit">
              <div class="mb-3">
                <label class="form-label">Tên nhà xuất bản</label>
                <input type="text" class="form-control" v-model="formData.TenNXB" required>
              </div>
              <div class="mb-3">
                <label class="form-label">Địa chỉ</label>
                <input type="text" class="form-control" v-model="formData.DiaChi">
              </div>
              <div class="alert alert-danger" v-if="error">{{ error }}</div>
              <div class="text-end">
                <button type="button" class="btn btn-secondary me-2" data-bs-dismiss="modal">Hủy</button>
                <button type="submit" class="btn btn-primary" :disabled="loading">
                  {{ loading ? 'Đang xử lý...' : 'Lưu' }}
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
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            Bạn có chắc chắn muốn xóa nhà xuất bản này?
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Hủy</button>
            <button type="button" class="btn btn-danger" @click="handleDelete" :disabled="loading">
              {{ loading ? 'Đang xử lý...' : 'Xóa' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Modal } from 'bootstrap';
import api from '../../services/api';

export default {
  name: 'PublisherManagement',
  data() {
    return {
      publishers: [],
      formData: {
        TenNXB: '',
        DiaChi: ''
      },
      selectedPublisher: null,
      isEditing: false,
      loading: false,
      error: null,
      publisherModal: null,
      deleteModal: null
    };
  },
  async mounted() {
    await this.loadPublishers();
    this.publisherModal = new Modal(document.getElementById('publisherModal'));
    this.deleteModal = new Modal(document.getElementById('deleteModal'));
  },
  methods: {
    async loadPublishers() {
      try {
        const response = await api.getAllPublishers();
        this.publishers = response.data;
      } catch (error) {
        console.error('Lỗi khi tải danh sách nhà xuất bản:', error);
      }
    },
    showAddModal() {
      this.isEditing = false;
      this.formData = { TenNXB: '', DiaChi: '' };
      this.error = null;
      this.publisherModal.show();
    },
    showEditModal(publisher) {
      this.isEditing = true;
      this.selectedPublisher = publisher;
      this.formData = { ...publisher };
      this.error = null;
      this.publisherModal.show();
    },
    async handleSubmit() {
      this.loading = true;
      this.error = null;
      try {
        if (this.isEditing) {
          await api.updatePublisher(this.selectedPublisher._id, this.formData);
        } else {
          await api.createPublisher(this.formData);
        }
        await this.loadPublishers();
        this.publisherModal.hide();
      } catch (error) {
        this.error = error.response?.data?.message || 'Đã có lỗi xảy ra';
      } finally {
        this.loading = false;
      }
    },
    confirmDelete(publisher) {
      this.selectedPublisher = publisher;
      this.deleteModal.show();
    },
    async handleDelete() {
      this.loading = true;
      try {
        await api.deletePublisher(this.selectedPublisher._id);
        await this.loadPublishers();
        this.deleteModal.hide();
      } catch (error) {
        alert(error.response?.data?.message || 'Đã có lỗi xảy ra');
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>
