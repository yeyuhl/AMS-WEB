<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">用户身份码</label>
        <el-input v-model="query.personId" clearable placeholder="用户身份码" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">考勤位置</label>
        <el-input v-model="query.location" clearable placeholder="考勤位置" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <label class="el-form-item-label">用户名称</label>
        <el-input v-model="query.name" clearable placeholder="用户名称" style="width: 185px;" class="filter-item"
                  @keyup.enter.native="crud.toQuery"/>
        <date-range-picker
          v-model="query.time"
          start-placeholder="timeStart"
          end-placeholder="timeStart"
          class="date-item"
        />
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
        >上传
        </el-button>
      </crudOperation>
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                 :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="考勤时间">
            <el-date-picker v-model="form.time" type="datetime" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="用户身份码">
            <el-input v-model="form.personId" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="考勤位置">
            <el-input v-model="form.location" style="width: 370px;"/>
          </el-form-item>
          <el-form-item label="用户名称">
            <el-input v-model="form.name" style="width: 370px;"/>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>

      <el-dialog :visible.sync="dialogVisible" width="500px">
        <el-form size="small" label-width="80px">
          <el-form-item label="上传图片">
            <el-upload
              ref="upload"
              action=""
              accept=".png,.jpg,.jpeg"
              :auto-upload="false"
              :file-list="fileList"
              :http-request="uploadImages"
              :before-upload="beforeFileUpload"
            >
              <div class="eladmin-upload"><i class="el-icon-upload"/>添加图片</div>
              <div slot="tip" class="el-upload__tip">图片大小不超10MB</div>
            </el-upload>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="info" @click="dialogVisible=false">取消</el-button>
          <el-button type="primary" @click="manualUpload">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;"
                @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55"/>
        <el-table-column prop="personId" label="用户身份码"/>
        <el-table-column prop="name" label="用户名称"/>
        <el-table-column prop="time" label="考勤时间"/>
        <el-table-column prop="location" label="考勤位置"/>
        <el-table-column v-if="checkPer(['admin','faceAttendance:edit','faceAttendance:del'])" label="操作"
                         width="150px" align="center">
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
import crudFaceAttendance from '@/api/faceAttendance'
import CRUD, {presenter, header, form, crud} from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from "@/components/DateRangePicker/index.vue";
import axios from "axios";
import {getToken} from "@/utils/auth";


const defaultForm = {time: null, attendanceId: null, personId: null, location: null, name: null}
export default {
  name: 'FaceAttendance',
  components: {DateRangePicker, pagination, crudOperation, rrOperation, udOperation},
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({
      title: '考勤',
      url: 'api/faceAttendance',
      idField: 'attendanceId',
      sort: 'attendanceId,desc',
      crudMethod: {...crudFaceAttendance}
    })
  },
  data() {
    return {
      dialogVisible: false,
      fileList: [],
      permission: {
        add: ['', 'faceAttendance:add'],
        edit: ['', 'faceAttendance:edit'],
        del: ['', 'faceAttendance:del']
      },
      rules: {},
      queryTypeOptions: [
        {key: 'personId', display_name: '用户身份码'},
        {key: 'location', display_name: '考勤位置'},
        {key: 'name', display_name: '用户名称'}
      ]
    }
  },
  methods: {
    beforeFileUpload(file) {
      let isLt2M = true
      isLt2M = file.size / 1024 / 1024 < 10
      if (!isLt2M) {
        this.loading = false
        this.$message.error('上传文件大小不能超过10MB!')
      }
      return isLt2M
    },
    uploadImages(content) {
      let formData = new FormData()
      formData.append('file', content.file)
      let url = 'http://localhost:8000/api/faceAttendance/uploadImages'
      axios.post(url, formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
          'Authorization': getToken()
        }
      }).then(res => {
        this.crud.notify(res.data, CRUD.NOTIFICATION_TYPE.SUCCESS)
        this.fileList = [];
      }).catch(err => {
        this.crud.notify(err.response.data, CRUD.NOTIFICATION_TYPE.ERROR)
      })
    },
    manualUpload() {
      this.$refs.upload.submit()
      this.dialogVisible = false
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
