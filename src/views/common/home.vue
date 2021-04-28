<!-- 个人中心 -->
<template>
  <div class="center">
    <el-row :gutter="8">
      <el-col :span="6">
        <div class="item-content">
          <div class="avator">
            <div class="avator-img">
              <img src="~@/assets/img/avator.png" alt="">
            </div>
          </div>
          <p class="base-info">用户名：{{baseUserInfo.username}}</p>
          <p class="base-info">手机号: {{baseUserInfo.mobile}}</p>
          <p class="base-info">邮箱： {{baseUserInfo.email}}</p>
          <div>
            <el-button type="primary" @click="addOrUpdateHandle()">修改资料</el-button>
            <el-button type="primary" @click="updatePasswordHandle()">更改密码</el-button>
          </div>
        </div>
      </el-col>
      <el-col :span="18">
        <div class="item-content">
          <div class="right-title">健康信息 <span class="tips"><i class="el-icon-info" style="margin-right: 10px"></i>配置健康信息，可为您更好的个性化推荐运动</span></div>
          <div class="right-form">
            <el-form ref="form" :model="formData" label-width="100px">
              <el-form-item label="身高(cm)">
                <el-input-number v-model.number="formData.uheight" :min="10" class="form-width"></el-input-number>
              </el-form-item>
              <el-form-item label="体重(kg)">
                <el-input-number v-model.number="formData.uweight" :min="1" class="form-width"></el-input-number>
              </el-form-item>
              <!-- <el-form-item label="年龄">
                <el-input-number v-model.number="formData.uage" :min="1" class="form-width"></el-input-number>
              </el-form-item> -->
              <!-- <el-form-item label="健康度">
                <el-select v-model="formData.uhealth" placeholder="请选择健康度" class="form-width">
                  <el-option label="健康" value="健康"></el-option>
                  <el-option label="良好" value="良好"></el-option>
                  <el-option label="一般" value="一般"></el-option>
                  <el-option label="较差" value="较差"></el-option>
                </el-select>
              </el-form-item> -->
              <el-form-item label="兴趣">
                <el-checkbox-group v-model="formData.uhobbyLable">
                  <el-checkbox v-for="(x, i) in hobbyList" :key="i" :label="x" name="uhobbyLable"></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
              <el-form-item label="疾病状况">
                <el-checkbox-group v-model="formData.uhealth">
                  <el-checkbox v-for="(x, i) in healthList" :key="i" :label="x" name="uhealth"></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="onSubmit">保存设置</el-button>
              </el-form-item>
            </el-form>
          </div>
        </div>
      </el-col>
    </el-row>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getUserFunc"></add-or-update>
    <!-- 弹窗, 修改密码 -->
    <update-password v-if="updatePassowrdVisible" ref="updatePassowrd"></update-password>
    <!-- 兴趣标签 -->
  </div>
</template>

<script>
import AddOrUpdate from './user-add-or-update'
import updatePassword from '@/views/main-navbar-update-password.vue'
import { mapState } from 'vuex';
export default {
  props: {},
  data () {
    return {
      addOrUpdateVisible: false,
      baseUserInfo: {},
      updatePassowrdVisible: false,
      formData: {
        id: 0,
        uheight: '',
        uweight: '',
        uage: '',
        uhealth: [],
        uhobbyLable: []
      },
      hobbyList: [],
      healthList: []
    }
  },
  components: {
    AddOrUpdate,
    updatePassword
  },
  computed: {
    ...mapState({
      getUserInfo: state => state.user
    })
  },
  methods: {
    // 保存配置
    onSubmit() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysuserinfo/${!this.formData.id ? 'save' : 'update'}`),
        method: 'post',
        data: this.$http.adornData({
          'id': this.formData.id || undefined,
          'uage': this.formData.uage,
          'uhealth': this.formData.uhealth.join(','),
          'uheight': this.formData.uheight,
          'uhobbyLable': this.formData.uhobbyLable.join(','),
          'uweight': this.formData.uweight,
          'uid': this.getUserInfo.id,
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '操作成功',
            type: 'success',
            duration: 1500,
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    // 修改密码
    updatePasswordHandle () {
      this.updatePassowrdVisible = true
      this.$nextTick(() => {
        this.$refs.updatePassowrd.init()
      })
    },
    // 新增 / 修改
    addOrUpdateHandle () {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(this.getUserInfo.id)
      })
    },
    getUserFunc() {
      this.$http({
        url: this.$http.adornUrl(`/sys/user/info`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.baseUserInfo = data.user;
        }
      })
    },
    getLabelFunc() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysprojectinfo/getProjectLable`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.hobbyList = data.lables;
        }
      })
    },
    getillLabelFunc() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysprojectinfo/getIllLable`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.healthList = data.lables;
        }
      })
    },
    getConfigFunc() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysuserinfo/info/byUser`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          if(data.sysUserInfo) {
            this.formData.id = data.sysUserInfo.id;
            if(data.sysUserInfo.uage) {
              this.formData.uage = data.sysUserInfo.uage;
            }
            if(data.sysUserInfo.uheight) {
              this.formData.uheight = data.sysUserInfo.uheight;
            }
            if(data.sysUserInfo.uweight) {
              this.formData.uweight = data.sysUserInfo.uweight;
            }
            if(data.sysUserInfo.uhobbyLable) {
              this.formData.uhobbyLable = data.sysUserInfo.uhobbyLable.split(',');
            }
            if(data.sysUserInfo.uhealth) {
              this.formData.uhealth = data.sysUserInfo.uhealth.split(',');
            }
          }
        }
      })
    }
  },
  watch: {},
  mounted () {
    this.getUserFunc()
    this.getLabelFunc()
    this.getillLabelFunc()
    this.getConfigFunc()
  },
  created () {}
}
</script>

<style scoped lang="scss">
.center{
  background-color: #eee;
}

.item-content{
  height: calc(100vh - 135px);
  background-color: #fff;
  overflow: auto;
}

.avator{
  padding: 20px;
}

.avator-img{
  height: 150px;
  width: 150px;
  margin: 0 auto;
  img{
    width: 150px;
    height: 150px;
    border-radius: 50%;
  }
}

.base-info{
  margin-bottom: 20px;
  font-size: 16px;
  border-bottom: 1px dashed rgb(30, 25, 25);
  padding: 5px;
}
.right-title{
  padding: 10px 10px;
  font-size: 18px;
  border-bottom: 1px solid #eee;
}

.right-form{
  padding: 10px;
}

.form-width{
  width: 400px;
}

.tips{
  color: orange;
  font-size: 18px;
  padding: 10px 100px;
}
</style>