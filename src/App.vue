<template>
  <div id="app">
    <h1>文件传输工具链</h1>
    <div class="container">
      <FileUpload @file-uploaded="handleFileUpload" />
      <FileList :files="files" :password="password" @refresh="fetchFiles" />
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
      apiBaseUrl: process.env.VUE_APP_API_BASE_URL || 'http://ddns.curesky.site:7878'
    }
  },
  methods: {
    async handleFileUpload(file) {
      try {
        const formData = new FormData()
        formData.append('paste', file)
        
        const response = await fetch(`${this.apiBaseUrl}/save?password=${this.password}&filename=${encodeURIComponent(file.name)}`, {
          method: 'POST',
          body: formData
        })
        
        if (response.ok) {
          const fileId = await response.text()
          this.files.push({
            id: fileId,
            name: file.name,
            url: `${this.apiBaseUrl}/${fileId}?password=${this.password}`,
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
        
        const fileData = await response.json();
        this.files = fileData.map(file => ({
          id: file[0],
          name: file[2],
          url: `${this.apiBaseUrl}/${file[0]}?password=${this.password}`,
          size: file[1],
          uploadTime: new Date().toLocaleString()
        }));
        
        this.$message.success('文件列表获取成功');
      } catch (error) {
        console.error('获取文件列表失败:', error);
        this.$message.error('获取文件列表失败');
      }
    }
  },
  mounted() {
    this.fetchFiles();
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
