<template>
  <el-container>
    <el-header>
      <el-menu class="head-menu" mode="horizontal" @select="onSelectIndex">
        <el-menu-item index="index"><el-icon><HomeFilled /></el-icon> Index </el-menu-item>
        <el-menu-item index="asmr"> ASMR </el-menu-item>
        <el-menu-item index="acg"> ACG </el-menu-item>
        <el-menu-item index="else"> Else </el-menu-item>
      </el-menu>
    </el-header>

    <el-main>
      <el-button
        class="add-item-button"
        style="width: 100%" color="#626aef"
        @click="dialogSubmitState = true">
        Add URL
      </el-button>

      <el-table :data="tableData[pageIndex]" border>
        <el-table-column label="URL" width="300">
          <template #default="scope">
            <el-link type="primary" :href="scope.row.url" target="_blank"> {{ scope.row.title }} </el-link>
          </template>
        </el-table-column>
        <el-table-column prop="description" label="Description"></el-table-column>
        <el-table-column align="center" fixed="right" width="130">
          <template #default="scope">
            <el-button type="primary" :icon="Edit" @click="onClickEdit(scope.row)"/>
            <el-button color="#F56C6C" :icon="Delete" @click="onClickDelete(scope.row.id)"/>
          </template>
        </el-table-column>
      </el-table>
      
    </el-main>
  </el-container>

  <el-dialog class="submit-dialog" v-model="dialogSubmitState" title="Add URL" width="650">
    <el-form class="dialog-form" :model="formData.data" label-width="100px">
      <el-form-item label="Title" required>
        <el-input v-model="formData.data.title" placeholder="Input title here"></el-input>
      </el-form-item>
      <el-form-item label="URL" required>
        <el-input v-model="formData.data.url" placeholder="Input url here"></el-input>
      </el-form-item>
      <el-form-item label="Description">
        <el-input v-model="formData.data.description" placeholder="Input description here"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span>
        <el-button @click="dialogSubmitState = false"> Cancel </el-button>
        <el-button type="primary" @click="onSubmit"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog class="delete-dialog" v-model="dialogDeleteState" title="Are you sure to delete this url?" width="380">
    <template #footer>
      <span>
        <el-button @click="dialogDeleteState = false"> Cancel </el-button>
        <el-button type="primary" @click="onDelete()"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog class="edit-dialog" v-model="dialogEditState" title="Edit URL" width="650">
    <el-form class="dialog-form" :model="editData.data" label-width="100px">
      <el-form-item label="Title" required>
        <el-input v-model="editData.data.title" placeholder="Input title here"></el-input>
      </el-form-item>
      <el-form-item label="URL" required>
        <el-input v-model="editData.data.url" placeholder="Input url here"></el-input>
      </el-form-item>
      <el-form-item label="Description">
        <el-input v-model="editData.data.description" placeholder="Input description here"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span>
        <el-button @click="dialogEditState = false"> Cancel </el-button>
        <el-button type="primary" @click="onEdit"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref, reactive, onMounted, toRaw, readonly } from 'vue'
import { HomeFilled, Delete, Edit } from '@element-plus/icons-vue'
import axios from 'axios'

const pageArray = ['index', 'asmr', 'acg', 'else']
const pageIndex = ref('index')  // 页面索引
const dialogSubmitState = ref(false)    // 添加对话框
const dialogDeleteState = ref(false)    // 删除对话框
const dialogEditState = ref(false)      // 编辑对话框
let deleteColumnId = 0

const tableData = reactive({index: [], asmr: [], acg: [], else: []})  // 页面数据
const formData = reactive({ data: {} })    // 添加暂存数据
const editData = reactive({ data: {} })    // 编辑暂存数据

function onSelectIndex(index) {
  pageIndex.value = index
}

function onSubmit() {
  axios.post('/api/add_url?page=' + pageIndex.value, formData.data)
  .then(res => {
    formData.data = {}  // 置空
    dialogSubmitState.value = false
    getURL(pageIndex.value)
  })
  .catch(err => {
    alert(err)
  })
}

function onClickEdit(data) {
  // 深拷贝，防止修改页面值
  editData.data = JSON.parse(JSON.stringify(data))
  dialogEditState.value = true
}

function onEdit() {
  axios.post('/api/edit_url?page=' + pageIndex.value, editData.data)
  .then(res => {
    dialogEditState.value = false
    getURL(pageIndex.value)
  })
  .catch(err => {
    alert(err)
  })
}

function onClickDelete(id) {
  deleteColumnId = id
  dialogDeleteState.value = true
}

function onDelete() {
  var data = {};
  data.id = deleteColumnId;
  axios.post('/api/del_url?page=' + pageIndex.value, data)
  .then(res => {
    dialogDeleteState.value = false
    getURL(pageIndex.value)
  })
  .catch(err => {
    alert(err)
  })
}

function getURL(page) {
  axios.get('/api/get_url?page=' + page)
  .then(res => {
    tableData[page] = res.data.data
  })
  .catch(err => {
    alert(err)
  })
}

onMounted(() => { for (const page of pageArray) getURL(page) })
// watch(pageIndex.value, getURL(pageIndex.value))
</script>
  
<style scoped>

@media screen and (width <= 500px) {
  .delete-dialog {
    min-width: 280px !important;
  }
  .submit-dialog {
    min-width: 380px !important;
  }

  .edit-dialog {
    min-width: 380px !important;
  }
}

.add-item-button {
  margin-bottom: 20px;
}

.dialog-form {
  margin-right: 20px;
}
</style>