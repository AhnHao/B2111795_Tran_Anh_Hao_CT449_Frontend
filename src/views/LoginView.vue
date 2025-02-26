<template>
  <div class="login-container">
    <div class="card login-card">
      <div class="card-header text-center">
        <h3>Đăng nhập</h3>
      </div>
      <div class="card-body">
        <form @submit.prevent="handleLogin">
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
          <div class="alert alert-danger" v-if="error">
            {{ error }}
          </div>
          <div class="d-grid gap-2">
            <button type="submit" class="btn btn-primary" :disabled="loading">
              {{ loading ? 'Đang đăng nhập...' : 'Đăng nhập' }}
            </button>
          </div>
          <div class="text-center mt-3">
            <p class="mb-0">
              Chưa có tài khoản?
              <router-link to="/register">Đăng ký ngay</router-link>
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
  name: 'LoginView',
  data() {
    return {
      formData: {
        SoDienThoai: '',
        Password: ''
      },
      loading: false,
      error: null
    };
  },
  methods: {
    async handleLogin() {
      this.loading = true;
      this.error = null;
      try {
        const response = await api.login(this.formData);
        const { token, role, user } = response.data;
        
        localStorage.setItem('token', token);
        localStorage.setItem('userRole', role);
        localStorage.setItem('userData', JSON.stringify(user));

        if (role === 'staff') {
          this.$router.push('/staff');
        } else {
          this.$router.push('/reader');
        }
      } catch (error) {
        this.error = error.response?.data?.message || 'Đã có lỗi xảy ra';
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f8f9fa;
  padding: 15px;
}

.login-card {
  width: 100%;
  max-width: 400px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
}

.card-header {
  background-color: #fff;
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
  padding: 1.5rem;
}

.card-header h3 {
  margin: 0;
  color: #333;
}

.card-body {
  padding: 2rem;
}

.form-label {
  font-weight: 500;
  color: #555;
}

.form-control {
  padding: 0.75rem;
}

.btn-primary {
  padding: 0.75rem;
  font-weight: 500;
}

.alert {
  margin-bottom: 1rem;
}

@media (max-width: 576px) {
  .login-card {
    margin: 15px;
  }
  
  .card-body {
    padding: 1.5rem;
  }
}
</style>