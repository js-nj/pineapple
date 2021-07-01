<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
      <el-form-item label="品系" prop="beanName">
        <el-input v-model="dataForm.beanName" placeholder="品系名称, 如: 菠萝1号"></el-input>
      </el-form-item>
      <el-form-item label="树龄" prop="params">
        <el-input v-model="dataForm.params" placeholder="树龄"></el-input>
      </el-form-item>
      <el-form-item label="采集时间" prop="cronExpression">
        <el-date-picker
          v-model="dataForm.cronExpression"
          type="date"
          placeholder="采集时间">
        </el-date-picker>
        <!-- <el-input v-model="dataForm.cronExpression" placeholder="采集时间"></el-input> -->
      </el-form-item>
      <el-form-item label="品质" prop="remark">
        <el-select v-model="dataForm.remark" placeholder="品质">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        <!-- <el-input v-model="dataForm.remark" placeholder="品质"></el-input> -->
      </el-form-item>
      <el-form-item label="来源" prop="status">
        <el-input v-model="dataForm.status" placeholder="来源"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
         options:[{
          label:'高',
          value:'高'
        },{
          label:'中',
          value:'中'
        },{
          label:'低',
          value:'低'
        }],
        dataForm: {
          id: 0,
          beanName: '',
          params: '',
          cronExpression: '',
          remark: '',
          status: ''
        },
        dataRule: {
          beanName: [
            { required: true, message: '品系不能为空', trigger: 'blur' }
          ],
          params: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          remark: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          cronExpression: [
            { required: true, message: 'cron表达式不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: 'cron表达式不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/schedule/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.beanName = data.schedule.beanName
                this.dataForm.params = data.schedule.params
                this.dataForm.cronExpression = data.schedule.cronExpression
                this.dataForm.remark = data.schedule.remark
                this.dataForm.status = data.schedule.status
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/schedule/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'jobId': this.dataForm.id || undefined,
                'beanName': this.dataForm.beanName,
                'params': this.dataForm.params,
                'cronExpression': this.dataForm.cronExpression,
                'remark': this.dataForm.remark,
                'status': !this.dataForm.id ? undefined : this.dataForm.status
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
