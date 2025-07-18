<template>
  <div class="device-list-container">
    <el-card shadow="hover">
      <template #header>
        <div class="card-header">
          <div class="header-title">设备管理</div>
          <div class="header-actions">
            <el-dropdown>
              <el-button type="primary">
                <el-icon><Plus /></el-icon>添加设备
                <el-icon class="el-icon--right"><ArrowDown /></el-icon>
              </el-button>
              <template #dropdown>
                <el-dropdown-menu>
                  <el-dropdown-item @click="$router.push('/device/add')">手动添加</el-dropdown-item>
                  <el-dropdown-item>从模板创建</el-dropdown-item>
                  <el-dropdown-item>批量导入</el-dropdown-item>
                </el-dropdown-menu>
              </template>
            </el-dropdown>
          </div>
        </div>
      </template>

      <!-- 搜索区域 -->
      <el-form :model="queryParams" ref="queryForm" :inline="false" class="search-form filter-section">
        <el-row :gutter="20">
          <el-col :span="6">
            <el-form-item label="设备名称" prop="name">
              <el-input v-model="queryParams.name" placeholder="请输入设备名称" clearable />
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="设备类型" prop="type">
              <el-select v-model="queryParams.type" placeholder="全部类型" clearable style="width: 100%">
                <el-option label="摄像头" value="CAMERA" />
                <el-option label="传感器" value="SENSOR" />
                <el-option label="门禁" value="ACCESS" />
                <el-option label="其他" value="OTHER" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="设备状态" prop="status">
              <el-select v-model="queryParams.status" placeholder="全部状态" clearable style="width: 100%">
                <el-option label="在线" :value="1" />
                <el-option label="离线" :value="0" />
                <el-option label="故障" :value="2" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="快速创建" prop="template">
              <el-select v-model="queryParams.template" placeholder="请选择模板" clearable style="width: 100%">
                <el-option label="会议室摄像头" :value="1" />
                <el-option label="温湿度传感器" :value="2" />
                <el-option label="门禁控制器" :value="3" />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- 高级筛选 -->
        <el-collapse v-model="activeCollapse">
          <el-collapse-item name="advanced" class="advanced-filter">
            <template #title>
              <span><el-icon><Filter /></el-icon> 高级筛选</span>
            </template>
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="所属区域" prop="area">
                  <el-select v-model="queryParams.area" placeholder="全部区域" clearable style="width: 100%">
                    <el-option label="北区" value="NORTH" />
                    <el-option label="南区" value="SOUTH" />
                    <el-option label="东区" value="EAST" />
                    <el-option label="西区" value="WEST" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="设备分组" prop="group">
                  <el-select v-model="queryParams.group" placeholder="全部分组" clearable style="width: 100%">
                    <el-option label="安防监控" :value="1" />
                    <el-option label="环境监测" :value="2" />
                    <el-option label="门禁管理" :value="3" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="设备标签" prop="tags">
                  <el-select v-model="queryParams.tags" placeholder="选择标签" clearable multiple style="width: 100%">
                    <el-option label="重要" value="IMPORTANT" />
                    <el-option label="室外" value="OUTDOOR" />
                    <el-option label="室内" value="INDOOR" />
                    <el-option label="测试" value="TEST" />
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="设备协议" prop="protocol">
                  <el-select v-model="queryParams.protocol" placeholder="全部协议" clearable style="width: 100%">
                    <el-option label="RTSP" value="RTSP" />
                    <el-option label="ONVIF" value="ONVIF" />
                    <el-option label="MODBUS" value="MODBUS" />
                    <el-option label="HTTP" value="HTTP" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
          </el-collapse-item>
        </el-collapse>

        <div class="filter-actions">
          <el-button @click="resetQuery">重置</el-button>
          <el-button type="primary" @click="handleQuery">搜索</el-button>
        </div>
      </el-form>

      <!-- 设备统计 -->
      <div class="device-stats">
        <div class="device-stat-item">
          设备总数: <el-tag effect="plain">{{ deviceStats.total || 0 }}</el-tag>
        </div>
        <div class="device-stat-item">
          在线: <el-tag type="success" effect="plain">{{ deviceStats.online || 0 }}</el-tag>
        </div>
        <div class="device-stat-item">
          离线: <el-tag type="danger" effect="plain">{{ deviceStats.offline || 0 }}</el-tag>
        </div>
        <div class="device-stat-item">
          故障: <el-tag type="warning" effect="plain">{{ deviceStats.fault || 0 }}</el-tag>
        </div>
      </div>

      <!-- 视图切换 -->
      <div class="view-toggle">
        <el-radio-group v-model="viewType" size="small">
          <el-radio-button label="table">
            <el-icon><List /></el-icon> 列表
          </el-radio-button>
          <el-radio-button label="map">
            <el-icon><Location /></el-icon> 地图
          </el-radio-button>
          <el-radio-button label="chart">
            <el-icon><PieChart /></el-icon> 统计
          </el-radio-button>
        </el-radio-group>
        <div class="view-actions">
          <el-tooltip content="刷新数据">
            <el-button link type="primary" :icon="Refresh" circle @click="getList" />
          </el-tooltip>
          <el-dropdown>
            <el-button link type="primary" :icon="MoreFilled" circle />
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item>批量启用</el-dropdown-item>
                <el-dropdown-item>批量禁用</el-dropdown-item>
                <el-dropdown-item divided>批量删除</el-dropdown-item>
                <el-dropdown-item>导出设备</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
      </div>

      <!-- 表格视图 -->
      <div v-show="viewType === 'table'" class="table-view">
        <el-table
          v-loading="loading"
          :data="deviceList"
          border
          stripe
          style="width: 100%"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" width="55" align="center" />
          <el-table-column prop="name" label="设备名称" min-width="120" show-overflow-tooltip />
          <el-table-column prop="code" label="设备编码" width="120" show-overflow-tooltip />
          <el-table-column prop="type" label="设备类型" width="100" align="center">
            <template #default="{ row }">
              {{ getDeviceTypeText(row.type) }}
            </template>
          </el-table-column>
          <el-table-column prop="ipAddress" label="IP地址" width="120" show-overflow-tooltip />
          <el-table-column prop="area" label="所属区域" width="100" align="center" />
          <el-table-column label="设备标签" width="150">
            <template #default="{ row }">
              <div class="device-tags">
                <el-tag v-for="tag in row.tags" :key="tag" size="small" effect="plain" class="tag-item">
                  {{ tag }}
                </el-tag>
              </div>
            </template>
          </el-table-column>
          <el-table-column label="设备状态" width="100" align="center">
            <template #default="scope">
              <el-tag :type="getDeviceStatusType(scope.row.status)">
                {{ getDeviceStatusText(scope.row.status) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="最后心跳时间" width="180" align="center">
            <template #default="scope">
              {{ formatDateTime(scope.row.lastHeartbeatTime) }}
            </template>
          </el-table-column>
          <el-table-column label="操作" width="230" align="center" fixed="right">
            <template #default="scope">
              <el-button type="primary" link @click="handleDetail(scope.row)">
                详情
              </el-button>
              <el-button type="primary" link @click="handleEdit(scope.row)">
                编辑
              </el-button>
              <el-button type="primary" link @click="handleCheck(scope.row)">
                检测
              </el-button>
              <el-button type="danger" link @click="handleDelete(scope.row)">
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <!-- 地图视图 -->
      <div v-show="viewType === 'map'" class="map-view">
        <div id="device-map" style="height: 500px">
          <!-- 地图视图将在此渲染 -->
          <el-empty description="地图加载中..." />
        </div>
      </div>

      <!-- 统计视图 -->
      <div v-show="viewType === 'chart'" class="chart-view">
        <div id="device-chart" style="height: 500px">
          <!-- 统计图表将在此渲染 -->
          <el-empty description="图表加载中..." />
        </div>
      </div>

      <!-- 分页 -->
      <div v-show="viewType === 'table'" class="pagination-container">
        <el-pagination
          v-model:current-page="queryParams.page"
          v-model:page-size="queryParams.size"
          :page-sizes="[10, 20, 50, 100]"
          :total="total"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { formatDateTime } from '@/utils/date'
import { 
  Plus, ArrowDown, Filter, List, Location, PieChart, 
  Refresh, MoreFilled
} from '@element-plus/icons-vue'

const router = useRouter()

// 加载状态
const loading = ref(false)

// 高级筛选展开状态
const activeCollapse = ref([])

// 视图类型：table, map, chart
const viewType = ref('table')

// 设备列表数据
const deviceList = ref([])

// 总记录数
const total = ref(0)

// 选中的设备
const selectedDevices = ref([])

// 设备状态统计
const deviceStats = reactive({
  total: 0,
  online: 0,
  offline: 0,
  fault: 0
})

// 查询参数
const queryParams = reactive({
  page: 0,
  size: 10,
  name: '',
  type: '',
  status: '',
  template: '',
  area: '',
  group: '',
  tags: [],
  protocol: '',
  sort: 'id',
  direction: 'desc'
})

// 获取设备状态类型
const getDeviceStatusType = (status) => {
  const typeMap = {
    0: 'danger',
    1: 'success',
    2: 'warning'
  }
  return typeMap[status] || 'info'
}

// 获取设备状态文本
const getDeviceStatusText = (status) => {
  const textMap = {
    0: '离线',
    1: '在线',
    2: '故障'
  }
  return textMap[status] || '未知'
}

// 获取设备类型文本
const getDeviceTypeText = (type) => {
  const typeMap = {
    'CAMERA': '摄像头',
    'SENSOR': '传感器',
    'ACCESS': '门禁',
    'OTHER': '其他'
  }
  return typeMap[type] || type
}

// Mock数据：获取设备列表
const getList = async () => {
  loading.value = true
  try {
    // 模拟API调用
    setTimeout(() => {
      // Mock数据
      deviceList.value = [
        {
          id: 1,
          name: 'HD摄像头-会议室',
          code: 'CAM20230001',
          type: 'CAMERA',
          ipAddress: '192.168.1.101',
          area: '北区',
          tags: ['重要', '室内'],
          status: 1,
          lastHeartbeatTime: new Date().toISOString(),
          createdAt: '2023-05-10T10:00:00Z'
        },
        {
          id: 2,
          name: '温湿度传感器-办公区',
          code: 'SENS20230002',
          type: 'SENSOR',
          ipAddress: '192.168.1.102',
          area: '东区',
          tags: ['室内'],
          status: 1,
          lastHeartbeatTime: new Date().toISOString(),
          createdAt: '2023-05-12T14:30:00Z'
        },
        {
          id: 3,
          name: '门禁控制器-正门',
          code: 'ACC20230003',
          type: 'ACCESS',
          ipAddress: '192.168.1.103',
          area: '南区',
          tags: ['重要', '室外'],
          status: 0,
          lastHeartbeatTime: '2023-06-15T08:45:12Z',
          createdAt: '2023-05-15T09:20:00Z'
        },
        {
          id: 4,
          name: '360全景摄像头-大厅',
          code: 'CAM20230004',
          type: 'CAMERA',
          ipAddress: '192.168.1.104',
          area: '西区',
          tags: ['重要', '室内'],
          status: 2,
          lastHeartbeatTime: '2023-06-20T17:30:45Z',
          createdAt: '2023-05-18T16:10:00Z'
        },
        {
          id: 5,
          name: '烟雾传感器-档案室',
          code: 'SENS20230005',
          type: 'SENSOR',
          ipAddress: '192.168.1.105',
          area: '北区',
          tags: ['室内', '测试'],
          status: 1,
          lastHeartbeatTime: new Date().toISOString(),
          createdAt: '2023-05-20T11:15:00Z'
        }
      ]
      total.value = 5
      loading.value = false
    }, 500)
  } catch (error) {
    console.error('获取设备列表失败', error)
    loading.value = false
  }
}

// Mock数据：获取设备状态统计
const getDeviceStats = async () => {
  try {
    // 模拟API调用
    setTimeout(() => {
      deviceStats.online = 3
      deviceStats.offline = 1
      deviceStats.fault = 1
      deviceStats.total = deviceStats.online + deviceStats.offline + deviceStats.fault
    }, 300)
  } catch (error) {
    console.error('获取设备状态统计失败', error)
  }
}

// 查询按钮点击
const handleQuery = () => {
  queryParams.page = 0
  getList()
}

// 重置查询条件
const resetQuery = () => {
  queryParams.name = ''
  queryParams.type = ''
  queryParams.status = ''
  queryParams.template = ''
  queryParams.area = ''
  queryParams.group = ''
  queryParams.tags = []
  queryParams.protocol = ''
  handleQuery()
}

// 选择变更
const handleSelectionChange = (selection) => {
  selectedDevices.value = selection
}

// 每页条数变更
const handleSizeChange = (size) => {
  queryParams.size = size
  getList()
}

// 页码变更
const handleCurrentChange = (page) => {
  queryParams.page = page - 1
  getList()
}

// 查看详情
const handleDetail = (row) => {
  router.push(`/device/detail/${row.id}`)
}

// 编辑设备
const handleEdit = (row) => {
  router.push(`/device/edit/${row.id}`)
}

// 检测连接
const handleCheck = async (row) => {
  ElMessage({
    type: 'info',
    message: `正在检测设备 ${row.name} 的连接状态...`
  })
  // 模拟检测过程
  setTimeout(() => {
    ElMessage({
      type: 'success',
      message: `设备 ${row.name} 连接正常`
    })
  }, 1500)
}

// 删除设备
const handleDelete = (row) => {
  ElMessageBox.confirm(
    `确认删除设备"${row.name}"吗？`,
    '删除确认',
    {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    // 模拟删除操作
    setTimeout(() => {
      ElMessage({
        type: 'success',
        message: `设备"${row.name}"已删除`
      })
      getList()
    }, 500)
  }).catch(() => {})
}

// 生命周期钩子
onMounted(() => {
  getList()
  getDeviceStats()
})
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.header-title {
  font-size: 18px;
  font-weight: bold;
}
.search-form {
  margin-bottom: 20px;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 4px;
}
.filter-section .el-row {
  margin-bottom: 10px;
}
.advanced-filter {
  border-top: 1px dashed #ddd;
  padding-top: 15px;
  margin-top: 5px;
}
.filter-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 15px;
}
.device-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 20px;
  padding: 10px 15px;
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,.05);
}
.device-stat-item {
  display: flex;
  align-items: center;
  gap: 5px;
}
.view-toggle {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}
.view-actions {
  display: flex;
  gap: 10px;
}
.tag-item {
  margin: 2px;
}
.device-tags {
  display: flex;
  flex-wrap: wrap;
}
.map-view, .chart-view {
  margin-bottom: 20px;
  border-radius: 4px;
  overflow: hidden;
}
.pagination-container {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}
</style> 