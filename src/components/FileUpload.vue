<template>
  <div 
    class="upload-area"
    :class="{ 'drag-over': isDragOver }"
    @dragover.prevent="handleDragOver"
    @dragleave="handleDragLeave"
    @drop="handleDrop"
    @click="triggerFileInput"
  >
    <div class="upload-content">
      <p>拖拽文件到此处或点击上传</p>
      <input
        type="file"
        ref="fileInput"
        @change="handleFileSelect"
        style="display: none"
        multiple
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileUpload',
  data() {
    return {
      isDragOver: false
    }
  },
  methods: {
    handleDragOver(e) {
      e.preventDefault()
      this.isDragOver = true
    },
    handleDragLeave() {
      this.isDragOver = false
    },
    handleDrop(e) {
      e.preventDefault()
      this.isDragOver = false
      const files = e.dataTransfer.files
      this.processFiles(files)
    },
    triggerFileInput() {
      this.$refs.fileInput.click()
    },
    handleFileSelect(e) {
      const files = e.target.files
      this.processFiles(files)
    },
    processFiles(files) {
      for (let i = 0; i < files.length; i++) {
        this.$emit('file-uploaded', files[i])
      }
    }
  }
}
</script>

<style scoped>
.upload-area {
  border: 2px dashed #ccc;
  border-radius: 8px;
  padding: 40px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.upload-area:hover,
.upload-area.drag-over {
  border-color: #42b883;
  background-color: #f8f9fa;
}

.upload-content p {
  margin: 0;
  color: #666;
  font-size: 16px;
}
</style>
