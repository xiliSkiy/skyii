<template>
  <div class="device-types-container">
    <el-card shadow="hover">
      <template #header>
        <div class="card-header">
          <div class="header-title">设备类型管理</div>
          <div class="header-actions">
            <el-button type="primary" @click="handleAddType">
              <el-icon><Plus /></el-icon>添加类型
            </el-button>
          </div>
        </div>
      </template>

      <!-- 搜索及筛选 -->
      <div class="search-panel">
        <el-row :gutter="20">
          <el-col :span="8">
            <el-input
              v-model="queryParams.keyword"
              placeholder="请输入类型名称或编码"
              clearable
              @keyup.enter="handleSearch"
            >
              <template #prefix>
                <el-icon><Search /></el-icon>
              </template>
              <template #append>
                <el-button @click="handleSearch">搜索</el-button>
              </template>
            </el-input>
          </el-col>
          <el-col :span="16" class="text-right">
            <el-button @click="handleExport">
              <el-icon><Download /></el-icon>导出
            </el-button>
            <el-button @click="handleImport">
              <el-icon><Upload /></el-icon>导入
            </el-button>
          </el-col>
        </el-row>
      </div>

      <!-- 设备类型树形表格 -->
      <el-table
        v-loading="loading"
        :data="typeList"
        row-key="id"
        border
        :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
      >
        <el-table-column prop="name" label="类型名称" min-width="200">
          <template #default="{ row }">
            <div class="type-name">
              <el-icon v-if="row.icon === 'camera'"><VideoCamera /></el-icon>
              <el-icon v-else-if="row.icon === 'sensor'"><Odometer /></el-icon>
              <el-icon v-else-if="row.icon === 'access'"><Key /></el-icon>
              <el-icon v-else><Monitor /></el-icon>
              {{ row.name }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="code" label="类型编码" width="180" />
        <el-table-column prop="deviceCount" label="设备数量" width="120" align="center" />
        <el-table-column label="支持协议" width="250">
          <template #default="{ row }">
            <div class="protocol-badges">
              <el-tag
                v-for="protocol in row.protocols"
                :key="protocol"
                size="small"
                effect="plain"
                class="protocol-badge"
              >
                {{ protocol }}
              </el-tag>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="sort" label="排序" width="80" align="center" />
        <el-table-column label="操作" width="230" align="center" fixed="right">
          <template #default="{ row }">
            <el-button type="primary" link @click="handleView(row)">
              <el-icon><View /></el-icon>查看
            </el-button>
            <el-button type="primary" link @click="handleEdit(row)">
              <el-icon><Edit /></el-icon>编辑
            </el-button>
            <el-button type="primary" link @click="handleAddSubType(row)">
              <el-icon><Plus /></el-icon>添加子类型
            </el-button>
            <el-button type="danger" link @click="handleDelete(row)">
              <el-icon><Delete /></el-icon>删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 新增/编辑类型对话框 -->
    <el-dialog
      v-model="dialog.visible"
      :title="dialog.title"
      width="600px"
      destroy-on-close
      @closed="resetForm"
    >
      <el-form
        ref="typeFormRef"
        :model="typeForm"
        :rules="typeRules"
        label-width="100px"
      >
        <el-form-item label="类型名称" prop="name">
          <el-input v-model="typeForm.name" placeholder="请输入类型名称" />
        </el-form-item>
        <el-form-item label="类型编码" prop="code">
          <el-input v-model="typeForm.code" placeholder="请输入类型编码" />
        </el-form-item>
        <el-form-item label="上级类型" prop="parentId">
          <el-cascader
            v-model="typeForm.parentId"
            :options="typeOptions"
            :props="{ checkStrictly: true, value: 'id', label: 'name', emitPath: false }"
            placeholder="顶级类型"
            clearable
          />
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-select v-model="typeForm.icon" placeholder="请选择图标">
            <el-option label="摄像头" value="camera">
              <div class="icon-option">
                <el-icon><VideoCamera /></el-icon>
                <span>摄像头</span>
              </div>
            </el-option>
            <el-option label="传感器" value="sensor">
              <div class="icon-option">
                <el-icon><Odometer /></el-icon>
                <span>传感器</span>
              </div>
            </el-option>
            <el-option label="门禁" value="access">
              <div class="icon-option">
                <el-icon><Key /></el-icon>
                <span>门禁</span>
              </div>
            </el-option>
            <el-option label="通用设备" value="device">
              <div class="icon-option">
                <el-icon><Monitor /></el-icon>
                <span>通用设备</span>
              </div>
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="支持协议" prop="protocols">
          <el-select v-model="typeForm.protocols" multiple placeholder="请选择支持的协议">
            <el-option label="RTSP" value="RTSP" />
            <el-option label="ONVIF" value="ONVIF" />
            <el-option label="HTTP" value="HTTP" />
            <el-option label="HTTPS" value="HTTPS" />
            <el-option label="MQTT" value="MQTT" />
            <el-option label="MODBUS" value="MODBUS" />
          </el-select>
        </el-form-item>
        <el-form-item label="排序" prop="sort">
          <el-input-number v-model="typeForm.sort" :min="1" :max="999" />
        </el-form-item>
        <el-form-item label="描述" prop="description">
          <el-input v-model="typeForm.description" type="textarea" rows="3" placeholder="请输入描述" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button @click="dialog.visible = false">取消</el-button>
          <el-button type="primary" @click="submitForm">确认</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 查看类型详情对话框 -->
    <el-dialog
      v-model="detailDialog.visible"
      title="类型详情"
      width="600px"
      destroy-on-close
    >
      <el-descriptions :column="1" border>
        <el-descriptions-item label="类型名称">{{ currentType.name }}</el-descriptions-item>
        <el-descriptions-item label="类型编码">{{ currentType.code }}</el-descriptions-item>
        <el-descriptions-item label="上级类型">{{ currentType.parentName || '顶级类型' }}</el-descriptions-item>
        <el-descriptions-item label="设备数量">{{ currentType.deviceCount }}</el-descriptions-item>
        <el-descriptions-item label="排序">{{ currentType.sort }}</el-descriptions-item>
        <el-descriptions-item label="支持协议">
          <div class="protocol-badges">
            <el-tag
              v-for="protocol in currentType.protocols"
              :key="protocol"
              size="small"
              effect="plain"
              class="protocol-badge"
            >
              {{ protocol }}
            </el-tag>
          </div>
        </el-descriptions-item>
        <el-descriptions-item label="描述">{{ currentType.description || '无' }}</el-descriptions-item>
      </el-descriptions>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Plus, Edit, Delete, View, Search, Download, Upload, 
  VideoCamera, Odometer, Key, Monitor
} from '@element-plus/icons-vue'

// 查询参数
const queryParams = reactive({
  keyword: ''
})

// 加载状态
const loading = ref(false)

// 设备类型列表
const typeList = ref([])

// 类型选项（用于级联选择器）
const typeOptions = ref([])

// 当前编辑或查看的类型
const currentType = ref({})

// 对话框状态
const dialog = reactive({
  visible: false,
  title: '添加设备类型',
  type: 'add' // add, edit, addSub
})

// 详情对话框状态
const detailDialog = reactive({
  visible: false
})

// 表单引用
const typeFormRef = ref(null)

// 设备类型表单
const typeForm = reactive({
  id: null,
  name: '',
  code: '',
  parentId: null,
  icon: 'device',
  protocols: [],
  sort: 1,
  description: ''
})

// 表单验证规则
const typeRules = {
  name: [
    { required: true, message: '请输入类型名称', trigger: 'blur' },
    { min: 2, max: 50, message: '长度在 2 到 50 个字符', trigger: 'blur' }
  ],
  code: [
    { required: true, message: '请输入类型编码', trigger: 'blur' },
    { pattern: /^[A-Z0-9_]+$/, message: '只允许大写字母、数字和下划线', trigger: 'blur' }
  ]
}

// 获取设备类型列表
const getTypeList = () => {
  loading.value = true
  // 模拟 API 调用
  setTimeout(() => {
    // Mock 数据
    typeList.value = [
      {
        id: 1,
        name: '摄像头',
        code: 'CAMERA',
        deviceCount: 50,
        icon: 'camera',
        protocols: ['RTSP', 'ONVIF', 'HTTP'],
        sort: 1,
        description: '监控摄像设备',
        children: [
          {
            id: 11,
            name: '网络摄像头',
            code: 'CAMERA_NETWORK',
            deviceCount: 30,
            icon: 'camera',
            protocols: ['RTSP', 'ONVIF', 'HTTP'],
            sort: 1,
            description: '基于IP网络的摄像设备',
            children: [
              {
                id: 111,
                name: '球机',
                code: 'CAMERA_NETWORK_DOME',
                deviceCount: 15,
                icon: 'camera',
                protocols: ['RTSP', 'ONVIF'],
                sort: 1,
                description: '具有云台控制功能的网络摄像设备'
              },
              {
                id: 112,
                name: '枪机',
                code: 'CAMERA_NETWORK_BULLET',
                deviceCount: 10,
                icon: 'camera',
                protocols: ['RTSP', 'ONVIF'],
                sort: 2,
                description: '固定安装的网络摄像设备'
              },
              {
                id: 113,
                name: '半球',
                code: 'CAMERA_NETWORK_TURRET',
                deviceCount: 5,
                icon: 'camera',
                protocols: ['RTSP', 'ONVIF'],
                sort: 3,
                description: '小型网络半球摄像设备'
              }
            ]
          },
          {
            id: 12,
            name: '模拟摄像头',
            code: 'CAMERA_ANALOG',
            deviceCount: 10,
            icon: 'camera',
            protocols: ['BNC'],
            sort: 2,
            description: '传统模拟信号摄像设备'
          },
          {
            id: 13,
            name: '热成像摄像头',
            code: 'CAMERA_THERMAL',
            deviceCount: 5,
            icon: 'camera',
            protocols: ['RTSP', 'ONVIF'],
            sort: 3,
            description: '可在无光环境下工作的热成像设备'
          }
        ]
      },
      {
        id: 2,
        name: '传感器',
        code: 'SENSOR',
        deviceCount: 35,
        icon: 'sensor',
        protocols: ['MODBUS', 'MQTT', 'HTTP'],
        sort: 2,
        description: '各类环境传感器',
        children: [
          {
            id: 21,
            name: '温湿度传感器',
            code: 'SENSOR_TEMP_HUMI',
            deviceCount: 20,
            icon: 'sensor',
            protocols: ['MODBUS', 'MQTT'],
            sort: 1,
            description: '监测温度和湿度的传感设备'
          },
          {
            id: 22,
            name: '气体传感器',
            code: 'SENSOR_GAS',
            deviceCount: 10,
            icon: 'sensor',
            protocols: ['MODBUS'],
            sort: 2,
            description: '监测气体浓度的传感设备'
          },
          {
            id: 23,
            name: '烟雾传感器',
            code: 'SENSOR_SMOKE',
            deviceCount: 5,
            icon: 'sensor',
            protocols: ['MODBUS', 'HTTP'],
            sort: 3,
            description: '监测烟雾浓度的传感设备'
          }
        ]
      },
      {
        id: 3,
        name: '门禁',
        code: 'ACCESS',
        deviceCount: 15,
        icon: 'access',
        protocols: ['HTTP', 'HTTPS'],
        sort: 3,
        description: '门禁控制设备',
        children: [
          {
            id: 31,
            name: '门禁控制器',
            code: 'ACCESS_CONTROLLER',
            deviceCount: 10,
            icon: 'access',
            protocols: ['HTTP', 'HTTPS'],
            sort: 1,
            description: '门禁系统中的控制主机'
          },
          {
            id: 32,
            name: '读卡器',
            code: 'ACCESS_READER',
            deviceCount: 5,
            icon: 'access',
            protocols: ['Wiegand'],
            sort: 2,
            description: '用于读取门禁卡的设备'
          }
        ]
      }
    ]
    loading.value = false
    
    // 构建类型选项
    buildTypeOptions()
  }, 500)
}

// 构建类型选项（用于级联选择器）
const buildTypeOptions = () => {
  const options = []
  
  const buildOptions = (types, options) => {
    types.forEach(type => {
      const option = {
        id: type.id,
        name: type.name,
        children: []
      }
      
      if (type.children && type.children.length > 0) {
        buildOptions(type.children, option.children)
      }
      
      options.push(option)
    })
  }
  
  buildOptions(typeList.value, options)
  typeOptions.value = options
}

// 搜索
const handleSearch = () => {
  getTypeList()
}

// 添加设备类型
const handleAddType = () => {
  dialog.title = '添加设备类型'
  dialog.type = 'add'
  dialog.visible = true
}

// 编辑设备类型
const handleEdit = (row) => {
  dialog.title = '编辑设备类型'
  dialog.type = 'edit'
  Object.assign(typeForm, row)
  dialog.visible = true
}

// 添加子类型
const handleAddSubType = (row) => {
  dialog.title = '添加子类型'
  dialog.type = 'addSub'
  resetForm()
  typeForm.parentId = row.id
  dialog.visible = true
}

// 查看设备类型
const handleView = (row) => {
  currentType.value = row
  detailDialog.visible = true
}

// 删除设备类型
const handleDelete = (row) => {
  ElMessageBox.confirm(
    `确认删除设备类型"${row.name}"吗？删除后其下所有子类型将一并删除！`,
    '删除确认',
    {
      confirmButtonText: '确认',
      cancelButtonText: '取消',
      type: 'warning',
    }
  )
    .then(() => {
      // 模拟删除
      ElMessage({
        type: 'success',
        message: `设备类型"${row.name}"已删除`,
      })
      getTypeList()
    })
    .catch(() => {})
}

// 导出类型
const handleExport = () => {
  ElMessage({
    type: 'info',
    message: '导出设备类型功能开发中',
  })
}

// 导入类型
const handleImport = () => {
  ElMessage({
    type: 'info',
    message: '导入设备类型功能开发中',
  })
}

// 重置表单
const resetForm = () => {
  if (typeFormRef.value) {
    typeFormRef.value.resetFields()
  }
  
  Object.assign(typeForm, {
    id: null,
    name: '',
    code: '',
    parentId: null,
    icon: 'device',
    protocols: [],
    sort: 1,
    description: ''
  })
}

// 提交表单
const submitForm = async () => {
  if (!typeFormRef.value) return
  
  await typeFormRef.value.validate((valid) => {
    if (valid) {
      // 模拟提交
      setTimeout(() => {
        ElMessage({
          type: 'success',
          message: dialog.type === 'edit' ? '修改成功' : '添加成功',
        })
        dialog.visible = false
        getTypeList()
      }, 500)
    }
  })
}

// 初始化
onMounted(() => {
  getTypeList()
})
</script>

<style scoped>
.device-types-container {
  padding: 6px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-title {
  font-size: 18px;
  font-weight: bold;
}

.search-panel {
  margin-bottom: 20px;
}

.text-right {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.type-name {
  display: flex;
  align-items: center;
  gap: 6px;
}

.protocol-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.protocol-badge {
  margin-right: 0;
}

.icon-option {
  display: flex;
  align-items: center;
  gap: 8px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>