<!-- src/views/System/components/OperLogDialog.vue -->
<template>
  <el-dialog :model-value="visible" @update:model-value="(val) => $emit('update:visible', val)" title="操作日志详情" width="70%" :destroy-on-close="true">
    <el-card>
      <el-descriptions column="1" border>
        <!-- 核心筛选字段优先展示，与搜索栏对应 -->
        <el-descriptions-item label="请求URL">{{ logInfo.operUrl || '-' }}</el-descriptions-item>
        <el-descriptions-item label="操作人员">{{ logInfo.operName || '未知用户' }}</el-descriptions-item>
        <el-descriptions-item label="操作时间">{{ formatTime(logInfo.operTime) || '-' }}</el-descriptions-item>

        <!-- 基础信息 -->
        <el-descriptions-item label="日志ID">{{ logInfo.id || '-' }}</el-descriptions-item>
        <el-descriptions-item label="模块标题">{{ logInfo.title || '-' }}</el-descriptions-item>

        <!-- 网络相关信息 -->
        <el-descriptions-item label="操作IP">{{ logInfo.operIp || '-' }}</el-descriptions-item>
        <el-descriptions-item label="主机地址">{{ logInfo.hostAddress || '-' }}</el-descriptions-item>
        <el-descriptions-item label="操作地点">{{ logInfo.operLocation || '未知地点' }}</el-descriptions-item>

        <!-- 操作状态与请求信息 -->
        <el-descriptions-item label="操作状态">
          <el-tag v-if="logInfo.status === 0" type="success">成功</el-tag>
          <el-tag v-else type="danger">异常</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="请求方式">{{ logInfo.requestMethod || '-' }}</el-descriptions-item>

        <!-- 扩展信息（可折叠） -->
        <el-descriptions-item label="请求参数">
          <el-collapse>
            <el-collapse-item title="查看参数">
              <pre v-if="logInfo.operParam" class="param-pre">{{ formatJson(logInfo.operParam) }}</pre>
              <span v-else>无参数</span>
            </el-collapse-item>
          </el-collapse>
        </el-descriptions-item>
        <el-descriptions-item label="返回结果">
          <el-collapse>
            <el-collapse-item title="查看结果">
              <pre v-if="logInfo.jsonResult" class="param-pre">{{ formatJson(logInfo.jsonResult) }}</pre>
              <span v-else>无返回结果</span>
            </el-collapse-item>
          </el-collapse>
        </el-descriptions-item>

        <!-- 错误信息（异常时显示） -->
        <el-descriptions-item label="错误信息" v-if="logInfo.status === 1">
          <div class="error-message">{{ logInfo.errorMsg || '未知错误' }}</div>
        </el-descriptions-item>

        <!-- 性能信息 -->
        <el-descriptions-item label="消耗时间">{{ logInfo.costTime ? `${logInfo.costTime}ms` : '-' }}</el-descriptions-item>
      </el-descriptions>
    </el-card>

    <template #footer>
      <el-button @click="$emit('update:visible', false)">关闭</el-button>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue'
import { SysOperLog } from '@/api/interface'
import { format } from 'date-fns' // 假设使用date-fns处理时间格式

// 格式化JSON显示
const formatJson = (jsonStr: string) => {
  try {
    const jsonObj = JSON.parse(jsonStr)
    return JSON.stringify(jsonObj, null, 2)
  } catch (e) {
    return jsonStr
  }
}

// 格式化时间（与搜索栏时间格式保持一致）
const formatTime = (timeStr: string) => {
  if (!timeStr) return ''
  try {
    return format(new Date(timeStr), 'yyyy-MM-dd HH:mm:ss')
  } catch (e) {
    return timeStr
  }
}

// 定义props
defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  logInfo: {
    type: Object as () => SysOperLog.ResOperLogList,
    default: () => ({})
  }
})

// 定义emits
defineEmits(['update:visible'])
</script>

<style scoped>
.param-pre {
  background-color: #f5f7fa;
  padding: 10px;
  border-radius: 4px;
  overflow-x: auto;
  font-size: 12px;
  color: #333;
}

.error-message {
  color: #f56c6c;
  background-color: #fef0f0;
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #fbc4c4;
}

::v-deep .el-descriptions-item__content {
  word-break: break-all;
}

/* 优化空值显示样式 */
::v-deep .el-descriptions-item__content:empty::before {
  content: '-';
  color: #999;
}
</style>
