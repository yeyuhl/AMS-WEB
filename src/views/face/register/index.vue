<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">身份码</label>
        <el-input v-model="query.personId" clearable placeholder="身份码" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">用户名称</label>
        <el-input v-model="query.name" clearable placeholder="用户名称" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">所属公司</label>
        <el-input v-model="query.company" clearable placeholder="所属公司" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">黑名单</label>
        <el-input v-model="query.type" clearable placeholder="黑名单(0为白名单，1为黑名单)" style="width: 185px;"
                  class="filter-item" @keyup.enter.native="crud.toQuery"/>
        <rrOperation :crud="crud"/>
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission">
        <el-button
          slot="left"
          class="filter-item"
          size="mini"
          type="primary"
          icon="el-icon-upload"
          @click="dialogVisible = true"
        >批量注册
        </el-button>
      </crudOperation>
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                 :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="身份码" prop="personId">
            <el-input v-model="form.personId" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="用户名称">
            <el-input v-model="form.name" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="所属公司">
            <el-input v-model="form.company" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="黑名单" prop="type">
            <el-input v-model="form.type" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="文件名" prop="fileName">
            <template v-slot="scope">
              <el-input v-model="fileName" style="width: 370px;"/>
              <div slot="tip" class="el-upload__tip">文件名要和身份码一致</div>
            </template>
          </el-form-item>
          <el-form-item label="上传">
            <template v-slot="scope">
              <el-upload
                ref="upload"
                action=""
                :limit="1"
                :auto-upload="false"
                :file-list="fileList"
                :http-request="uploadImage"
                :before-upload="beforeImageUpload"
              >
                <div class="eladmin-upload"><i class="el-icon-upload"/>添加图片</div>
                <div slot="tip" class="el-upload__tip">图片大小不超10MB</div>
              </el-upload>
            </template>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="manualUpload2">确认</el-button>
        </div>
      </el-dialog>

      <el-dialog :visible.sync="dialogVisible" width="500px">
        <el-form size="small" label-width="80px">
          <el-form-item label="上传文件">
            <el-upload
              accept=".csv"
              action="http://localhost:8000/api/faceRegister/uploadCSV"
              :limit="1"
              :headers="headers"
              :file-list="csvList"
            >
              <div class="eladmin-upload"><i class="el-icon-upload"/>添加CSV文件</div>
            </el-upload>
          </el-form-item>
          <el-form-item label="上传图片">
            <el-upload
              ref="upload2"
              action=""
              accept=".png,.jpg,.jpeg"
              :auto-upload="false"
              :file-list="fileList"
              :http-request="uploadImages"
              :before-upload="beforeImageUpload"
            >
              <div class="eladmin-upload"><i class="el-icon-upload"/>添加图片</div>
              <div slot="tip" class="el-upload__tip">图片大小不超10MB，，并且图片名要和身份码一致</div>
            </el-upload>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="info" @click="dialogVisible=false">取消</el-button>
          <el-button type="primary" @click="manualUpload1">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;"
                @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55"/>
        <el-table-column prop="personId" label="身份码"/>
        <el-table-column prop="name" label="用户名称"/>
        <el-table-column prop="company" label="所属公司"/>
        <el-table-column prop="type" label="黑名单"/>
        <el-table-column prop="image" label="图片">
          <template v-slot="scope">
            <el-image
              :src="scope.row.image"
              fit="contain"
              lazy
              style="width: 100px; height: 100px"
            >
              <div slot="error">
                <i class="el-icon-document"/>
              </div>
            </el-image>
          </template>
        </el-table-column>
        <el-table-column v-if="checkPer(['admin','faceRegister:edit','faceRegister:del'])" label="操作" width="150px"
                         align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination/>
    </div>
  </div>
</template>

<script>
import crudFaceRegister from '@/api/faceRegister'
import CRUD, {crud, form, header, presenter} from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import {getToken} from "@/utils/auth";
import axios from "axios";

const defaultForm = {name: null, personId: null, company: null, image: null, type: null}
export default {
  name: 'FaceRegister',
  components: {pagination, crudOperation, rrOperation, udOperation},
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({
      title: '人脸注册',
      url: 'api/faceRegister',
      idField: 'personId',
      sort: 'personId,desc',
      crudMethod: {...crudFaceRegister}
    })
  },
  data() {
    return {
      headers: {'Authorization': getToken()},
      fileName: '',
      fileList: [],
      csvList: [],
      dialogVisible: false,
      permission: {
        add: ['admin', 'faceRegister:add'],
        edit: ['admin', 'faceRegister:edit'],
        del: ['admin', 'faceRegister:del']
      },
      rules: {
        personId: [
          {required: true, message: '身份码不能为空', trigger: 'blur'}
        ],
        type: [
          {required: true, message: '黑名单(0为白名单，1为黑名单)不能为空', trigger: 'blur'}
        ]
      },
      queryTypeOptions: [
        {key: 'name', display_name: '用户名称'},
        {key: 'personId', display_name: '身份码'},
        {key: 'company', display_name: '所属公司'},
        {key: 'type', display_name: '黑名单(0为白名单，1为黑名单)'}
      ]
    }
  },
  methods: {
    beforeImageUpload(file) {
      let isLt2M = true
      isLt2M = file.size / 1024 / 1024 < 10
      if (!isLt2M) {
        this.loading = false
        this.$message.error('上传文件大小不能超过10MB!')
      }
      return isLt2M
    },
    uploadImage(content) {
      let formData = new FormData()
      formData.append('file', content.file)
      formData.append('fileName', this.fileName)
      let url = 'http://localhost:8000/api/faceRegister/uploadImages'
      axios.post(url, formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
          'Authorization': getToken()
        }
      }).then(res => {
        this.crud.notify('上传成功', CRUD.NOTIFICATION_TYPE.SUCCESS)
        this.updateImageUrl(res)
      }).catch(err => {
        this.crud.notify(err.response.data, CRUD.NOTIFICATION_TYPE.ERROR)
      })
    },
    uploadImages(content) {
      let formData = new FormData()
      formData.append('file', content.file)
      let url = 'http://localhost:8000/api/faceRegister/uploadImages'
      axios.post(url, formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
          'Authorization': getToken()
        }
      }).then(res => {
        this.crud.notify('上传成功', CRUD.NOTIFICATION_TYPE.SUCCESS)
        this.fileList = []
        this.updateImageUrl(res)
      }).catch(err => {
        this.crud.notify(err.response.data, CRUD.NOTIFICATION_TYPE.ERROR)
      })
    },
    updateImageUrl(res) {
      let newFormData = new FormData()
      newFormData.append('str', res.data)
      let url = 'http://localhost:8000/api/faceRegister/updateImageUrl'
      axios.post(url, newFormData, {
        headers: {
          'Content-Type': 'multipart/form-data',
          'Authorization': getToken()
        }
      }).then(res => {
        this.crud.notify(res.data, CRUD.NOTIFICATION_TYPE.SUCCESS)
      }).catch(err => {
        this.crud.notify(err.response.data, CRUD.NOTIFICATION_TYPE.ERROR)
      })
    },
    manualUpload1() {
      //let len = this.$refs['upload1'].$children[0].fileList.length
      let len2 = this.$refs['upload2'].$children[0].fileList.length
      // if (len) {
      //   this.$refs.upload1.submit()
      // }
      if (len2) {
        this.$refs.upload2.submit()
      }
      this.csvList = []
      this.dialogVisible = false
    },
    manualUpload2() {
      this.crud.submitCU()
      this.$refs.upload.submit()
    },
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
