<template>
  <div class="login-page">
    <el-card class="login-card">
      <h2 class="title">AI 智能养老社区管理系统</h2>
      <p class="subtitle">管理端</p>
      <el-form ref="formRef" :model="form" :rules="rules" @keyup.enter="submit">
        <el-form-item prop="phone">
          <el-input v-model="form.phone" placeholder="手机号" size="large" :prefix-icon="Iphone" />
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="form.password" type="password" placeholder="密码" size="large" show-password :prefix-icon="Lock" />
        </el-form-item>
        <el-button type="primary" size="large" class="login-btn" :loading="loading" @click="submit">登 录</el-button>
      </el-form>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { Iphone, Lock } from '@element-plus/icons-vue'
import { useAuthStore } from '../../store/auth'

const auth = useAuthStore()
const router = useRouter()
const route = useRoute()

const formRef = ref()
const loading = ref(false)
const form = reactive({ phone: '13800000000', password: '' })
const rules = {
  phone: [{ required: true, message: '请输入手机号', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
}

async function submit() {
  await formRef.value.validate()
  loading.value = true
  try {
    await auth.login(form.phone, form.password)
    ElMessage.success('登录成功')
    router.push(route.query.redirect || '/dashboard')
  } catch (e) {
    // 错误信息已由 axios 拦截器提示
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.login-page {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #0b3d91, #1890ff);
}
.login-card {
  width: 380px;
  padding: 12px 12px 4px;
}
.title {
  text-align: center;
  font-size: 20px;
  margin-bottom: 4px;
}
.subtitle {
  text-align: center;
  color: #909399;
  margin-bottom: 20px;
}
.login-btn {
  width: 100%;
}
</style>
