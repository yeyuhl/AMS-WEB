<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">用户身份码</label>
        <el-input v-model="query.personId" clearable placeholder="用户身份码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">识别位置</label>
        <el-input v-model="query.location" clearable placeholder="识别位置" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.time"
          start-placeholder="timeStart"
          end-placeholder="timeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="身份码">
            <el-input v-model="form.personId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="识别时间" prop="time">
            <el-date-picker v-model="form.time" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="识别位置" prop="location">
            <el-input v-model="form.location" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="personId" label="用户身份码" />
        <el-table-column prop="time" label="识别时间" />
        <el-table-column prop="location" label="识别位置" />
        <el-table-column v-if="checkPer(['admin','faceRecognition:edit','faceRecognition:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudFaceRecognition from '@/api/faceRecognition'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from "@/components/DateRangePicker/index.vue";

const defaultForm = { recognitionId: null, personId: null, time: null, location: null }
export default {
  name: 'FaceRecognition',
  components: {DateRangePicker, pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '轨迹查询', url: 'api/faceRecognition', idField: 'recognitionId', sort: 'recognitionId,desc', crudMethod: { ...crudFaceRecognition }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'faceRecognition:add'],
        edit: ['admin', 'faceRecognition:edit'],
        del: ['admin', 'faceRecognition:del']
      },
      rules: {
        time: [
          { required: true, message: '识别时间不能为空', trigger: 'blur' }
        ],
        location: [
          { required: true, message: '识别位置不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'personId', display_name: '用户身份码' },
        { key: 'location', display_name: '识别位置' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
