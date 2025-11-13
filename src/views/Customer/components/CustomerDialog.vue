<template>
  <Dialog
    :model-value="dialogVisible"
    :title="dialogProps.title"
    :fullscreen="dialogProps.fullscreen"
    :max-height="dialogProps.maxHeight"
    :cancel-dialog="cancelDialog"
    width="50%"
  >
    <div :style="'width: calc(100% - ' + dialogProps.labelWidth! / 2 + 'px)'">
      <el-form
        ref="ruleFormRef"
        label-position="right"
        :label-width="dialogProps.labelWidth + 'px'"
        :rules="rules"
        :model="dialogProps.row"
        :disabled="dialogProps.isView"
        :hide-required-asterisk="dialogProps.isView"
      >
        <el-form-item label="客户名称" prop="name">
          <el-input v-model="dialogProps.row!.name" placeholder="请填写客户名称" clearable maxlength="20" show-word-limit></el-input>
        </el-form-item>
        <el-form-item label="客户手机" prop="phone">
          <el-input v-model="dialogProps.row!.phone" placeholder="请填写客户手机" clearable maxlength="11" show-word-limit></el-input>
        </el-form-item>
        <el-form-item label="客户邮箱" prop="email">
          <el-input v-model="dialogProps.row!.email" placeholder="请填写客户邮箱" clearable maxlength="50" show-word-limit></el-input>
        </el-form-item>
        <el-form-item label="客户地址" prop="address">
          <el-input v-model="dialogProps.row!.address" placeholder="请填写客户地址" clearable maxlength="50" show-word-limit></el-input>
        </el-form-item>
        <!-- 修复filterable属性：将字符串改为布尔值true，并使用placeholder设置提示文字 -->
        <el-form-item label="客户等级" prop="level">
          <el-select v-model="dialogProps.row!.level" filterable placeholder="请选择客户等级">
            <el-option v-for="item in Object.values(CustomerLevelList)" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="客户来源" prop="source">
          <el-select v-model="dialogProps.row!.source" filterable placeholder="请选择客户来源">
            <el-option v-for="item in Object.values(CustomerSourceList)" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="跟进状态" prop="followStatus">
          <el-select v-model="dialogProps.row!.followStatus" filterable placeholder="请选择客户跟进状态">
            <el-option v-for="item in Object.values(FollowUpStatusList)" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="是否为关键决策人" prop="isKeyDecisionMaker">
          <el-select v-model="dialogProps.row!.isKeyDecisionMaker" filterable placeholder="请选择客户是否为关键决策人">
            <el-option v-for="item in Object.values(IsKeyDecisionMakerList)" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
        <el-form-item label="客户性别" prop="gender">
          <el-select v-model="dialogProps.row!.gender" filterable placeholder="请选择客户性别">
            <el-option v-for="item in Object.values(GenderList)" :key="item.value" :label="item.label" :value="item.value" />
          </el-select>
        </el-form-item>
      </el-form>
    </div>
    <template #footer>
      <slot name="footer">
        <el-button @click="cancelDialog">取消</el-button>
        <el-button type="primary" v-show="!dialogProps.isView" @click="handleSubmit">确定</el-button>
      </slot>
    </template>
  </Dialog>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { ElMessage, FormInstance } from 'element-plus'
import { Dialog } from '@/components/Dialog'
import { CustomerLevelList, CustomerSourceList, FollowUpStatusList, GenderList, IsKeyDecisionMakerList } from '@/configs/enum'

interface DialogProps {
  title: string
  isView: boolean
  fullscreen?: boolean
  row: any
  labelWidth?: number
  maxHeight?: number | string
  api?: (params: any) => Promise<any>
  getTableList?: () => Promise<any>
}

const dialogVisible = ref(false)
const dialogProps = ref<DialogProps>({
  isView: false,
  title: '',
  row: {},
  labelWidth: 160,
  fullscreen: false,
  maxHeight: '500px'
})

// 接收父组件传过来的参数
const acceptParams = (params: DialogProps): void => {
  // 修复参数合并逻辑，避免覆盖父组件传入的方法
  dialogProps.value = {
    ...dialogProps.value,
    ...params,
    // 保留原有的方法（如果父组件没传的话）
    api: params.api || dialogProps.value.api,
    getTableList: params.getTableList || dialogProps.value.getTableList
  }
  // 合并row数据，避免覆盖
  dialogProps.value.row = { ...dialogProps.value.row, ...params.row }
  dialogVisible.value = true
}

defineExpose({
  acceptParams
})

const rules = reactive({
  name: [{ required: true, message: '请输入客户名称', trigger: 'blur' }],
  phone: [
    { required: true, message: '请输入客户手机号', trigger: 'blur' },
    {
      pattern: /^1[3-9]\d{9}$/,
      message: '手机号格式不正确',
      trigger: 'blur'
    }
  ],
  email: [
    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
    {
      pattern: /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/,
      message: '邮箱格式不正确',
      trigger: 'blur'
    }
  ],
  level: [{ required: true, message: '客户级别不能为空', trigger: 'blur' }],
  source: [{ required: true, message: '客户来源不能为空', trigger: 'blur' }]
})

const ruleFormRef = ref<FormInstance>()

const handleSubmit = () => {
  ruleFormRef.value!.validate(async (valid) => {
    if (!valid) return
    try {
      // 深拷贝避免修改原对象
      const submitData = { ...dialogProps.value.row }
      delete submitData['updateTime']
      delete submitData['createTime']

      // 检查api是否存在
      if (!dialogProps.value.api) {
        ElMessage.error('未配置提交接口')
        return
      }

      await dialogProps.value.api(submitData)
      ElMessage.success({ message: `${dialogProps.value.title}成功！` })

      // 安全调用getTableList，避免不存在时报错
      if (dialogProps.value.getTableList && typeof dialogProps.value.getTableList === 'function') {
        await dialogProps.value.getTableList()
      }

      dialogVisible.value = false
      ruleFormRef.value!.resetFields()
      cancelDialog(true)
    } catch (error) {
      console.error('提交失败:', error)
      ElMessage.error('操作失败，请重试')
    }
  })
}

const cancelDialog = (isClean?: boolean) => {
  dialogVisible.value = false
  const condition = ['查看', '编辑']
  if (condition.includes(dialogProps.value.title) || isClean) {
    dialogProps.value.row = {}
    if (ruleFormRef.value) {
      ruleFormRef.value.resetFields()
    }
  }
}
</script>
