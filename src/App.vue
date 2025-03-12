<template>
  <div class="app-container">
    <!-- 顶部输入框和提交按钮 -->
    <div class="input-container">
      <el-input
        v-model="newTask"
        placeholder="请输入任务"
        class="task-input"
        clearable
      >
        <template #append>
          <el-button type="primary" @click="addTask" :icon="Plus">提交</el-button>
        </template>
      </el-input>
    </div>

    <!-- 底部任务展示区域 -->
    <div class="tasks-container">
      <!-- 正在进行中的任务 -->
      <el-card class="task-list">
        <template #header>
          <h2>正在进行中的任务</h2>
        </template>
        <draggable
          v-model="ongoingTasks"
          group="tasks"
          item-key="id"
          @end="onDragEnd"
          class="draggable-list"
        >
          <template #item="{ element: task }">
            <div class="task-item">
              <el-tag
                :type="task.isLoading ? 'info' : 'primary'"
                closable
                @close="deleteTask(task.id, 'ongoing')"
                class="custom-tag"
              >
                <span @click="openEditModal(task)">{{ task.name }}</span>
                <el-icon class="action-icon" @click="completeTask(task.id)">
                  <CircleCheck />
                </el-icon>
                <el-icon class="action-icon" @click="openEditModal(task)">
                  <Edit />
                </el-icon>
              </el-tag>
              <el-icon v-if="task.isLoading" class="is-loading">
                <Loading />
              </el-icon>
            </div>
          </template>
        </draggable>
      </el-card>

      <!-- 已完成的任务 -->
      <el-card class="task-list">
        <template #header>
          <h2>已完成的任务</h2>
        </template>
        <div v-for="task in completedTasks" :key="task.id" class="task-item">
          <el-tag type="success" closable @close="deleteTask(task.id, 'completed')" class="custom-tag">
            <span>{{ task.name }}</span>
          </el-tag>
        </div>
      </el-card>
    </div>

    <!-- 修改任务的弹窗 -->
    <el-dialog v-model="editModalVisible" title="修改任务" width="30%">
      <el-input v-model="editTaskName" placeholder="请输入新的任务名称" />
      <template #footer>
        <el-button @click="editModalVisible = false">取消</el-button>
        <el-button type="primary" @click="saveEdit">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { ElMessage } from 'element-plus';
import { Plus, CircleCheck, Edit, Loading } from '@element-plus/icons-vue';
import draggable from 'vuedraggable';

// 任务数据
const tasks = ref([]);
const newTask = ref('');
const editModalVisible = ref(false);
const editTaskName = ref('');
const currentTaskId = ref(null);

// 计算属性：正在进行中的任务
const ongoingTasks = computed({
  get: () => tasks.value.filter((task) => !task.completed),
  set: (newValue) => {
    tasks.value = [
      ...newValue,
      ...tasks.value.filter((task) => task.completed),
    ];
  },
});

// 计算属性：已完成的任务
const completedTasks = computed(() =>
  tasks.value.filter((task) => task.completed)
);

// 添加任务
const addTask = () => {
  if (newTask.value.trim() === '') {
    ElMessage.warning('任务内容不能为空');
    return;
  }
  tasks.value.push({
    id: Date.now(),
    name: newTask.value,
    completed: false,
    isLoading: true, // 模拟加载状态
  });
  newTask.value = '';
};

// 删除任务
const deleteTask = (taskId, type) => {
  tasks.value = tasks.value.filter((task) => task.id !== taskId);
  ElMessage.success(`已删除${type === 'ongoing' ? '进行中' : '已完成'}任务`);
};

// 完成任务
const completeTask = (taskId) => {
  const task = tasks.value.find((task) => task.id === taskId);
  if (task) {
    task.completed = true;
    task.isLoading = false;
    ElMessage.success('任务已完成');
  }
};

// 打开编辑弹窗
const openEditModal = (task) => {
  currentTaskId.value = task.id;
  editTaskName.value = task.name;
  editModalVisible.value = true;
};

// 保存编辑
const saveEdit = () => {
  const task = tasks.value.find((task) => task.id === currentTaskId.value);
  if (task) {
    task.name = editTaskName.value;
    editModalVisible.value = false;
    ElMessage.success('任务已修改');
  }
};

// 拖拽结束事件
const onDragEnd = () => {
  ElMessage.success('任务顺序已更新');
};
</script>

<style scoped>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.input-container {
  margin-bottom: 20px;
}

.task-input {
  width: 100%;
}

.tasks-container {
  display: flex;
  gap: 20px;
}

.task-list {
  flex: 1;
}

.draggable-list {
  min-height: 100px;
}

.task-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  padding: 8px;
  background-color: #f9fafc;
  border-radius: 4px;
  cursor: move;
}

.custom-tag {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  cursor: pointer;
}

.custom-tag span {
  flex: 1;
}

.action-icon {
  margin-left: 8px;
  cursor: pointer;
  color: #666;
}

.action-icon:hover {
  color: #409eff;
}

.is-loading {
  margin-left: 10px;
  animation: rotate 2s linear infinite;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>