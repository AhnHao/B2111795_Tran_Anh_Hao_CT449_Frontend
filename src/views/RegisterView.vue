<template>
  <div class="register-container">
    <div class="card register-card">
      <div class="card-header text-center">
        <h3>Đăng ký tài khoản</h3>
        <div class="btn-group mt-3" role="group">
          <button 
            type="button" 
            class="btn" 
            :class="isStaff ? 'btn-primary' : 'btn-outline-primary'"
            @click="isStaff = true"
          >
            Nhân viên
          </button>
          <button 
            type="button" 
            class="btn" 
            :class="!isStaff ? 'btn-primary' : 'btn-outline-primary'"
            @click="isStaff = false"
          >
            Độc giả
          </button>
        </div>
      </div>
      <div class="card-body">
        <form @submit.prevent="handleRegister">
          <div class="mb-3">
            <label for="name" class="form-label">Họ và tên</label>
            <input
              type="text"
              class="form-control"
              id="name"
              v-model="formData.HoTen"
              required
            />
          </div>
          <div class="mb-3">
            <label for="phone" class="form-label">Số điện thoại</label>
            <input
              type="text"
              class="form-control"
              id="phone"
              v-model="formData.SoDienThoai"
              required
            />
          </div>
          <div class="mb-3">
            <label for="password" class="form-label">Mật khẩu</label>
            <input
              type="password"
              class="form-control"
              id="password"
              v-model="formData.Password"
              required
            />
          </div>
          <div class="mb-3">
            <label for="address" class="form-label">Địa chỉ</label>
            <input
              type="text"
              class="form-control"
              id="address"
              v-model="formData.DiaChi"
            />
          </div>
          <div v-if="isStaff" class="mb-3">
            <label for="position" class="form-label">Chức vụ</label>
            <input
              type="text"
              class="form-control"
              id="position"
              v-model="formData.ChucVu"
              required
            />
          </div>
          <div v-if="!isStaff" class="mb-3">
            <label for="birthdate" class="form-label">Ngày sinh</label>
            <input
              type="date"
              class="form-control"
              id="birthdate"
              v-model="formData.NgaySinh"
              required
            />
          </div>
          <div class="alert alert-danger" v-if="error">
            {{ error }}
          </div>
          <div class="d-grid gap-2">
            <button type="submit" class="btn btn-primary" :disabled="loading">
              {{ loading ? 'Đang đăng ký...' : 'Đăng ký' }}
            </button>
          </div>
          <div class="text-center mt-3">
            <p class="mb-0">
              Đã có tài khoản?
              <router-link to="/login">Đăng nhập ngay</router-link>
            </p>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../services/api';

export default {
  name: 'RegisterView',
  data() {
    return {
      isStaff: false,
      formData: {
        HoTen: '',
        SoDienThoai: '',
        Password: '',
        DiaChi: '',
        ChucVu: '',
        NgaySinh: ''
      },
      loading: false,
      error: null
    };
  },
  methods: {
    async handleRegister() {
      this.loading = true;
      this.error = null;
      try {
        const endpoint = this.isStaff ? '/auth/register/staff' : '/auth/register/reader';
        const data = {
          ...this.formData,
          [this.isStaff ? 'HoTenNV' : 'HoTen']: this.formData.HoTen
        };

        if (this.isStaff) {
          delete data.NgaySinh;
          delete data.HoTen;
        } else {
          delete data.ChucVu;
          delete data.HoTenNV;
        }

        await api.post(endpoint, data);
        this.$router.push('/login');
      } catch (error) {
        this.error = error.response?.data?.message || 'Đã có lỗi xảy ra';
      } finally {
        this.loading = false;
      }
    }
  },
  watch: {
    isStaff() {
      this.formData = {
        HoTen: '',
        SoDienThoai: '',
        Password: '',
        DiaChi: '',
        ChucVu: '',
        NgaySinh: ''
      };
      this.error = null;
    }
  }
};
</script>

<style scoped>
.register-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f8f9fa;
  padding: 15px;
}

.register-card {
  width: 100%;
  max-width: 400px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.card-header {
  background-color: #fff;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
  padding: 1.5rem;
}

.card-body {
  padding: 2rem;
}

.btn-group {
  width: 100%;
}

.btn-group .btn {
  flex: 1;
}
</style>
