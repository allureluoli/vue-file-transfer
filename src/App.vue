<template>
  <div id="app">
    <h1>文件传输工具</h1>
    <div class="container">
      <FileUpload @file-uploaded="handleFileUpload" />
      <FileList :files="files" @refresh="fetchFiles" />
    </div>
  </div>
</template>
<script>
import FileUpload from './components/FileUpload.vue'
import FileList from './components/FileList.vue'
let pw = prompt('输入密码进行验证')
export default {
  name: 'App',
  components: {
    FileUpload,
    FileList
  },
  data() {
    return {
      files: [],
      password: pw,
      apiBaseUrl: process.env.VUE_APP_API_BASE_URL || 'http://ddns.curesky.site:7878' // 添加后端地址
    }
  },
  methods: {
    async handleFileUpload(file) {
      try {
        const formData = new FormData()
        formData.append('paste', file)
        
        const response = await fetch(`${this.apiBaseUrl}/save?password=${this.password}`, {
          method: 'POST',
          body: formData
        })
        
        if (response.ok) {
          const fileUrl = await response.text()
          this.files.push({
            name: file.name,
            url: `${this.apiBaseUrl}/${fileUrl}`, // 确保URL完整
            size: file.size,
            uploadTime: new Date().toLocaleString()
          })
        } else {
          console.error('文件上传失败')
        }
      } catch (error) {
        console.error('上传错误:', error)
      }
    },
    async fetchFiles() {
      try {
        const response = await fetch(`${this.apiBaseUrl}/list?password=${this.password}`);
        
        if (response.status === 401) {
          this.$message.error('密码错误或未授权');
          return;
        }
        
        if (!response.ok) {
          throw new Error('获取文件列表失败');
        }
        
        const fileNames = await response.json();
        // 将文件名数组转换为包含完整URL的文件对象数组
        this.files = fileNames.map(name => ({
          name,
          url: `${this.apiBaseUrl}/download/${name}`, // 根据你的后端路由调整
          size: 0, // 需要后端提供文件大小信息
          uploadTime: '' // 需要后端提供上传时间
        }));
        
        this.$message.success('文件列表获取成功');
      } catch (error) {
        console.error('获取文件列表失败:', error);
        this.$message.error('获取文件列表失败');
      }
    }
  },
  mounted() {
    this.fetchFiles(); // 组件挂载时获取文件列表
  }
}

</script>


<style scoped>
.container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}
</style>
