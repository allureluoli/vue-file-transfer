<template>
  <div class="file-list">
    <div class="list-header">
      <h2>文件列表</h2>
      <button @click="$emit('refresh')" class="refresh-btn">刷新</button>
    </div>
    
    <div v-if="files.length === 0" class="empty-state">
      暂无文件
    </div>
    
    <ul v-else class="files">
      <li 
        v-for="file in files" 
        :key="file.id"
        class="file-item"
        @click="downloadFile(file)"
        @contextmenu.prevent="showContextMenu($event, file)"
      >
        <span class="file-name">{{ file.name }}</span>
        <span class="file-size">{{ formatFileSize(file.size) }}</span>
        <span class="file-time">{{ file.uploadTime }}</span>
      </li>
    </ul>

    <div 
      v-if="contextMenu.visible"
      class="context-menu"
      :style="{ top: contextMenu.y + 'px', left: contextMenu.x + 'px' }"
      @click="hideContextMenu"
    >
      <div class="menu-item" @click="copyFileUrl">复制文件地址</div>
      <div class="menu-item" @click="downloadSelectedFile">下载文件</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileList',
  props: {
    files: {
      type: Array,
      default: () => []
    },
    password: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      contextMenu: {
        visible: false,
        x: 0,
        y: 0,
        selectedFile: null
      },
      // 正确的基础URL
      baseUrl: 'http://ddns.curesky.site:7878'
    }
  },
  methods: {
    // 文件大小格式化函数
    formatFileSize(bytes) {
      if (bytes === 0 || bytes === null || bytes === undefined) return '0 B'
      
      const sizes = ['B', 'KB', 'MB', 'GB', 'TB']
      const i = Math.floor(Math.log(bytes) / Math.log(1024))
      
      if (i === 0) return bytes + ' ' + sizes[i]
      
      return (bytes / Math.pow(1024, i)).toFixed(2) + ' ' + sizes[i]
    },

    // 生成文件URL - 修复路径问题
    generateFileUrl(fileId) {
      // 移除多余的 /download/ 路径，直接使用文件ID
      return `${this.baseUrl}/${fileId}?password=${this.password}`
    },

    // 显示右键菜单
    showContextMenu(event, file) {
      this.contextMenu = {
        visible: true,
        x: event.clientX,
        y: event.clientY,
        selectedFile: file
      }
    },

    // 隐藏右键菜单
    hideContextMenu() {
      this.contextMenu.visible = false
    },

    // 下载文件
    downloadFile(file) {
      const url = this.generateFileUrl(file.id)
      window.open(url, '_blank')
    },

    // 下载选中的文件
    downloadSelectedFile() {
      if (this.contextMenu.selectedFile) {
        this.downloadFile(this.contextMenu.selectedFile)
      }
      this.hideContextMenu()
    },

    async copyFileUrl() {
      if (!this.contextMenu.selectedFile) {
        this.hideContextMenu()
        return
      }
      
      const fileUrl = this.generateFileUrl(this.contextMenu.selectedFile.id)
      
      try {
        const success = await this.copyToClipboard(fileUrl)
        if (success) {
          this.showMessage('文件地址已复制到剪贴板')
        } else {
          this.showMessage(`复制失败，请手动复制：${fileUrl}`, 'error')
        }
      } catch (error) {
        console.error('复制出错:', error)
        this.showMessage(`复制出错，请手动复制：${fileUrl}`, 'error')
      }
      
      this.hideContextMenu()
    },
    
    async copyToClipboard(text) {
      // 方法1: 现代 Clipboard API (优先)
      if (navigator.clipboard && typeof navigator.clipboard.writeText === 'function') {
        try {
          await navigator.clipboard.writeText(text)
          return true
        } catch (error) {
          console.warn('Clipboard API 失败，尝试传统方法:', error)
          // 继续尝试传统方法
        }
      }
      
      // 方法2: 传统 execCommand 方法
      return this.copyWithExecCommand(text)
    },
    
    copyWithExecCommand(text) {
      try {
        const textArea = document.createElement('textarea')
        textArea.value = text
        textArea.style.position = 'fixed'
        textArea.style.left = '-9999px'
        textArea.style.top = '0'
        textArea.setAttribute('readonly', '')
        textArea.style.opacity = '0'
        
        document.body.appendChild(textArea)
        
        // 兼容移动设备
        if (navigator.userAgent.match(/ipad|iphone|android/i)) {
          textArea.contentEditable = true
          textArea.readOnly = true
          const range = document.createRange()
          range.selectNodeContents(textArea)
          const selection = window.getSelection()
          selection.removeAllRanges()
          selection.addRange(range)
          textArea.setSelectionRange(0, 999999)
        } else {
          textArea.select()
        }
        
        const successful = document.execCommand('copy')
        document.body.removeChild(textArea)
        return successful
      } catch (error) {
        console.error('传统复制方法失败:', error)
        return false
      }
    },
    
    showMessage(message, type = 'success') {
      // 消息提示兼容处理
      try {
        if (this.$message && typeof this.$message[type] === 'function') {
          this.$message[type](message)
        } else if (this.$message && this.$message.success) {
          this.$message.success(message)
        } else {
          alert(message)
        }
      } catch (error) {
        alert(message) // 最终降级到 alert
      }
    }
  }
}
</script>

<style scoped>
.file-list {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.refresh-btn {
  padding: 8px 16px;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.refresh-btn:hover {
  background-color: #369c70;
}

.empty-state {
  text-align: center;
  color: #999;
  padding: 40px;
}

.files {
  list-style: none;
  padding: 0;
  margin: 0;
}

.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eee;
  cursor: pointer;
}

.file-item:hover {
  background-color: #f5f5f5;
}

.file-name {
  flex: 2;
  text-align: left;
}

.file-size,
.file-time {
  flex: 1;
  text-align: right;
  color: #666;
  font-size: 14px;
}

.context-menu {
  position: fixed;
  background: white;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  z-index: 1000;
}

.menu-item {
  padding: 8px 16px;
  cursor: pointer;
}

.menu-item:hover {
  background-color: #f0f0f0;
}
</style>
