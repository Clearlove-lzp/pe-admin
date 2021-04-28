<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="运动名称" prop="projectName">
        <el-input v-model="dataForm.projectName" placeholder="运动名称"></el-input>
      </el-form-item>
      <el-form-item label="运动类型" prop="projectType">
        <el-input v-model="dataForm.projectType" placeholder="运动类型"></el-input>
      </el-form-item>
      <el-form-item label="运动标签" prop="projectLable">
        <el-input v-model="dataForm.projectLable" placeholder="格式：xx,xx,xx,xx"></el-input>
      </el-form-item>
      <!-- <el-form-item label="适合年龄" prop="projectAges">
        <el-input v-model="dataForm.projectAges" placeholder="格式：xx~xx"></el-input>
      </el-form-item> -->
      <el-form-item label="疾病限制" prop="illLable">
        <el-input v-model="dataForm.illLable" placeholder="格式：xx,xx,xx,xx"></el-input>
      </el-form-item>
      <el-form-item label="运动描述" prop="projectDesc">
        <el-input v-model="dataForm.projectDesc" placeholder="运动描述"></el-input>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">启用</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="上传图片" size="mini" prop="pic">
        <el-upload
          class="avatar-uploader"
          :headers="{token: token}"
          :action="uploadUrl"
          accept=".jpg,.png"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="beforeAvatarUpload">
          <img v-if="imageUrl" :src="imageUrl" class="avatar">
          <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { isEmail, isMobile } from '@/utils/validate'
  import Vue from 'vue'
  export default {
    data () {
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          projectName: '',
          projectType: '',
          projectLable: '',
          illLable: '',
          projectAges: '',
          projectDesc: '',
          status: 1,
          imageId: ""
        },
        dataRule: {
          projectName: [
            { required: true, message: '运动名称不能为空', trigger: 'blur' }
          ],
          projectType: [
            { required: true, message: '运动类型不能为空', trigger: 'blur' }
          ],
          projectLable: [
            { required: true, message: '运动标签不能为空', trigger: 'blur' }
          ],
          // illLable: [
          //   { required: true, message: '疾病限制不能为空', trigger: 'blur' }
          // ],
          projectAges: [
            { required: true, message: '适合年龄不能为空', trigger: 'blur' }
          ],
          projectDesc: [
            { required: true, message: '运动描述不能为空', trigger: 'blur' },
          ]
        },
        imageUrl: "",
        uploadUrl: "",
        token: ""
      }
    },
    methods: {
      init (info) {
        this.dataForm.id = info.id || 0
        this.visible = true
        if(this.dataForm.id) {
          this.dataForm.projectName = info.projectName
          this.dataForm.projectType = info.projectType
          this.dataForm.projectLable = info.projectLable
          this.dataForm.projectAges = info.projectAges
          this.dataForm.projectDesc = info.projectDesc
          this.dataForm.illLable = info.illLable
          this.dataForm.imageId = info.imageId
          this.dataForm.status = Number(info.status)
          this.imageUrl = info.imageUrl ? info.imageUrl : ""
        }else{
          this.dataForm = {
            id: 0,
            projectName: '',
            projectType: '',
            projectLable: '',
            illLable: '',
            projectAges: '',
            projectDesc: '',
            status: 1,
            imageId: ''
          }
        }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/sysprojectinfo/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectName': this.dataForm.projectName,
                'projectType': this.dataForm.projectType,
                'projectLable': this.dataForm.projectLable,
                'projectDesc': this.dataForm.projectDesc,
                'projectAges': this.dataForm.projectAges,
                'illLable': this.dataForm.illLable,
                'status': this.dataForm.status,
                'imageId': this.dataForm.imageId,
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                })
                this.visible = false
                this.$emit('refreshDataList')
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      handleAvatarSuccess(res, file) {
        console.log(res)
        console.log(file)
        if(res.code !== 0) {
          return this.$message.warning(res.msg)
        }
        this.dataForm.imageId = res.data.id;
        this.imageUrl = res.data.url;
      },
      beforeAvatarUpload(file) {

      }
    },
    mounted() {
      if(process.env.NODE_ENV === 'production') {
        this.uploadUrl = "http://139.186.167.136:10003/renren-fast/file/uploadHW"
      }else{
        this.uploadUrl = '/proxyApi/renren-fast/file/uploadHW'
      }
      this.token = Vue.cookie.get('token');
    }
  }
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
