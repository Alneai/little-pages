<template>
  <el-container>
    <el-header>
      <el-menu class="head-menu" mode="horizontal" @select="onSelect">
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
      <el-table :data="tableData" border>
        <el-table-column label="URL" width="300">
          <template #default="scope">
            <el-link type="primary" :href="scope.row.url" target="_blank"> {{ scope.row.title }} </el-link>
          </template>
        </el-table-column>
        <el-table-column prop="description" label="Description"></el-table-column>
        <el-table-column align="center" fixed="right" width="75">
          <template #default="scope">
            <el-button color="#F56C6C" :icon="Delete" @click="onClickDelete(scope.row.title)"/>
          </template>
        </el-table-column>
      </el-table>
      
    </el-main>
  </el-container>
  <el-dialog class="submit-dialog" v-model="dialogSubmitState" title="Add URL" width="650">
    <el-form class="submit-form" :model="formData" label-width="100px">
      <el-form-item label="Title" required>
        <el-input v-model="formData.title" placeholder="Input title here"></el-input>
      </el-form-item>
      <el-form-item label="URL" required>
        <el-input v-model="formData.url" placeholder="Input url here"></el-input>
      </el-form-item>
      <el-form-item label="Description">
        <el-input v-model="formData.description" placeholder="Input description here"></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogSubmitState = false"> Cancel </el-button>
        <el-button type="primary" @click="onSubmit"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>
  <el-dialog class="delete-dialog" v-model="dialogDeleteState" title="Are you sure to delete this url?" width="380">
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogDeleteState = false"> Cancel </el-button>
        <el-button type="primary" @click="onDelete()"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>
</template>
  
<script setup>
import { ref, reactive, onMounted, watch } from 'vue'
import { HomeFilled, Delete } from '@element-plus/icons-vue'
import axios from 'axios'

const page = ref('index')
const tableData = ref('')
const dialogSubmitState = ref(false)
const dialogDeleteState = ref(false)
const deleteTitle = ref('')

const formData = reactive({title: '', url: '', description: ''})

function onSelect(index) {
  page.value = index
}

function onSubmit() {
  axios.post('/api/add_url/' + page.value, formData)
  .then(res => {
    formData.title = ''
    formData.url = ''
    formData.description = ''
    dialogSubmitState.value = false
    getURL()
  })
  .catch(err => {
    alert(err)
  })
}

function onClickDelete(title) {
  dialogDeleteState.value = true
  deleteTitle.value = title
}

function onDelete() {
  var data = {};
  data.title = deleteTitle.value;
  axios.post('/api/del_url/' + page.value, data)
  .then(res => {
    dialogDeleteState.value = false
    getURL()
  })
  .catch(err => {
    alert(err)
  })
}

function getURL() {
  axios.get('/api/get_url/' + page.value)
  .then(res => {
    tableData.value = res.data.data
  })
  .catch(err => {
    alert(err)
  })
}

onMounted(getURL)
watch(page, getURL)
</script>
  
<style scoped>

@media screen and (width <= 500px) {
  .delete-dialog {
    min-width: 280px !important;
  }
  .submit-dialog {
    min-width: 380px !important;
  }
}

.add-item-button {
  margin-bottom: 20px;
}

.submit-form {
  margin-right: 20px;
}
</style>