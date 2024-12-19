<template>
  <el-row :gutter="0">
    <el-col :span="11">
      <el-button :icon="DocumentAdd" type="primary" @click="dialogFormVisible = true">新增</el-button>
      <el-button :icon="Printer" @click="exportExcel('用户信息', 'userTable')">导出</el-button>
    </el-col>
  </el-row>
  <el-row>
    <el-col>
      <el-table :data="tableData" border stripe ref="tableRef" id="userTable">
        <el-table-column label="操作" width="164">
          <template #default="scope">
            <el-button size="small" :icon="Edit" @click="handleEdit(scope.$index, scope.row)">
              编辑
            </el-button>
            <el-popconfirm title="确定删除吗？" @confirm="handleDelete(scope.$index, scope.row)">
              <template #reference>
                <el-button size="small" type="danger" :icon="Delete">
                  删除
                </el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
        <el-table-column prop="userId" label="用户ID" width="300" />
        <el-table-column prop="name" label="姓名" width="120" />
        <el-table-column prop="sex" label="性别" width="80" :formatter="sexFormatter" />
        <el-table-column prop="date" label="入职日期" width="120" sortable />
        <el-table-column prop="address" label="居住地址" width="360" />
      </el-table>
    </el-col>
  </el-row>

  <el-dialog v-model="dialogFormVisible" :title="form.userId != '' ? '修改' : '新增'" width="500" draggable
    :close-on-click-modal="false">
    <el-form :model="form" :rules="rules" status-icon ref="ruleFormRef">
      <el-form-item label="姓名" :label-width="formLabelWidth" prop="name">
        <el-input v-model="form.name" placeholder="请输入名称" autocomplete="off" style="width: 300px;" />
      </el-form-item>
      <el-form-item label="性别" :label-width="formLabelWidth" prop="sex">
        <el-select v-model="form.sex" placeholder="请选择性别" style="width: 220px;">
          <el-option label="男" value="man" />
          <el-option label="女" value="girl" />
        </el-select>
      </el-form-item>
      <el-form-item label="入职日期" :label-width="formLabelWidth" prop="date">
        <el-date-picker v-model="form.date" type="date" value-format="YYYY-MM-DD" placeholder="请选择入职日期" />
      </el-form-item>
      <el-form-item label="居住地址" :label-width="formLabelWidth" prop="address">
        <el-input v-model="form.address" style="width: 300px" :rows="2" type="textarea" placeholder="请输入居住地址" />
      </el-form-item>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="resetForm(ruleFormRef)">取消</el-button>
        <el-button type="primary" @click="submitForm(ruleFormRef)">
          确定
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref, toRaw } from 'vue'
import { Delete, Edit, Printer, RefreshRight, DocumentAdd } from '@element-plus/icons-vue'
import type { FormInstance, FormRules, TableInstance } from 'element-plus'
import { v4 as uuidv4 } from 'uuid'
import FileSaver from 'file-saver'
import * as XLSX from 'xlsx'

interface User {
  userId: string
  date: string
  name: string
  sex: string
  address: string
}

const tableData: User[] = reactive([])

let dialogFormVisible = ref(false)
const formLabelWidth = '120px'
const ruleFormRef = ref<FormInstance>()
const tableRef = ref<TableInstance>()
const form = reactive<User>({
  userId: '',
  name: '',
  sex: '',
  date: '',
  address: '',
})

const rules = reactive<FormRules<User>>({
  name: [
    { required: true, message: '姓名不能为空，请输入', trigger: 'blur' },
  ],
  sex: [
    {
      required: true,
      message: '性别不能为空，请选择',
      trigger: 'change',
    },
  ],
  date: [
    {
      type: 'date',
      required: true,
      message: '入职日期不能为空，请选择',
      trigger: 'change',
    },
  ],
  address: [
    { required: true, message: '居住地址不能为空，请输入', trigger: 'blur' },
  ],
})

const submitForm = async (formEl: FormInstance | undefined) => {
  if (!formEl) return
  await formEl.validate((valid, fields) => {
    if (valid) {
      dialogFormVisible.value = false
      if (form.userId === '') {
        form.userId = uuidv4()
        tableData.push({ ...form })
      } else {
        const userData: User[] = tableData.filter(user => user.userId === form.userId);
        if (userData.length > 0) {
          userData[0].name = form.name
          userData[0].sex = form.sex
          userData[0].date = form.date
          userData[0].address = form.address
        }
      }
      localStorage.setItem("userList", JSON.stringify(toRaw(tableData)))
      formEl.resetFields()
    } else {
      console.log('error submit!', fields)
    }
  })
}

const resetForm = (formEl: FormInstance | undefined) => {
  if (!formEl) return
  dialogFormVisible.value = false
  formEl.resetFields()
}

const handleEdit = (index: number, row: User) => {
  console.log(index, row)
  form.userId = row.userId
  form.name = row.name
  form.sex = row.sex
  form.date = row.date
  form.address = row.address
  dialogFormVisible.value = true
}
const handleDelete = (index: number, row: User) => {
  tableData.splice(index, 1)
  localStorage.setItem("userList", JSON.stringify(toRaw(tableData)))
}

const sexFormatter = (row: any, column: any, cellValue: any, index: number) => {
  if (cellValue === 'man') {
    return '男'
  }
  if (cellValue === 'girl') {
    return '女'
  }
}

onMounted: {
  const users = localStorage.getItem("userList");
  if (users != null) {
    const userList = JSON.parse(users)
    for (const index in userList) {
      tableData.push(userList[index])
    }
  }
}

//导出Excel
const exportExcel = (name: String, id: String) => {
  var wb = XLSX.utils.table_to_book(document.querySelector('#' + id))
  const cols = wb.Sheets.Sheet1['!cols'];
  if (cols) {
    cols[0] = { hidden: true }// 隐藏操作列
  }
  var wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' })
  try {
    FileSaver.saveAs(new Blob([wbout], { type: 'application/octet-stream' }), name + '.xlsx')
  } catch (e) { if (typeof console !== 'undefined') console.log(e, wbout) }
  return wbout
}
</script>

<style>
.el-row {
  margin: 5px;
}
</style>