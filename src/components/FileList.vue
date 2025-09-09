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
        :key="file.name"
        class="file-item"
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
    }
  },
  data() {
    return {
      contextMenu: {
        visible: false,
        x: 0,
        y: 0,
        selectedFile: null
      }
    }
  },
  methods: {
    showContextMenu(event, file) {
      this.contextMenu = {
        visible: true,
        x: event.clientX,
        y: event.clientY,
        selectedFile: file
      }
    },
    hideContextMenu() {
      this.contextMenu.visible = false
    },
    copyFileUrl() {
      if (this.contextMenu.selectedFile) {
        navigator.clipboard.writeText(this.contextMenu.selectedFile.url)
          .then(() => {
            alert('文件地址已复制到剪贴板')
          })
          .catch(err => {
            console.error('复制失败:', err)
          })
      }
      this.hideContextMenu()
    },
    formatFileSize(bytes) {
      if (bytes === 0) return '0 B'
      const k = 1024
      const sizes = ['B', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }
  },
  mounted() {
    document.addEventListener('click', this.hideContextMenu)
  },
  beforeUnmount() {
    document.removeEventListener('click', this.hideContextMenu)
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
