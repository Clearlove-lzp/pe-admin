<!-- 推荐管理 -->
<template>
  <div class="pe-container">
    <el-row>
      <el-col :span="16">
        <marquee class="tips"><i class="el-icon-info" style="margin-right: 10px"></i>以下是根据您的健康信息为您推荐的运动，如若未配置，则默认按照管理员创建运动信息数据先后排列</marquee>
        <div class="pe-list">
          <div class="pe-item" v-for="(x, i) in dataArr" :key="i">
            <img
              style="width: 200px; height: 200px"
              :src="x.imageUrl ? x.imageUrl : '~@/assets/img/avator.png'"></img>
            <div class="bottom-info">
              <div class="title bottom">{{x.projectName}}</div>
              <div class="detail bottom" @click="showDetail(x.id)">详情</div>
              <img style="width: 20px; height: 20px; position: absolute; right: 5px; bottom: 6px" src="~@/assets/img/hand-tap.png"></img>
            </div>
            <img v-if="i < 3" src="~@/assets/img/jian.png" style="width: 50px;height: 50px; position: absolute;left: 0; top: 0" alt="">
            <div style="width: 200px;height: 30px; lineHeight: 30px;position: absolute; bottom: 35px; left: 0;textAlign: center; background: #4c484875; color: #fff;">匹配度：{{x.matchingRate}}</div>
          </div>
        </div>
      </el-col>
      <el-col :span="8">
        <div class="echart-container">
          <div ref="echartBar" id="echartBar" class="chart-box"></div>
        </div>
      </el-col>
    </el-row>
    
    <el-dialog
      title="详情"
      :visible.sync="dialogVisible"
      width="60%">
      <div class="dialog-container">
        <div class="imitate-table">
          <table>
            <tr>
              <td>运动类型</td>
              <td>{{detailObj.projectType}}</td>
            </tr>
            <tr>
              <td>运动名称</td>
              <td>
                <div style="position: relative">
                  <img style="width: 200px; height: 200px" :src="detailObj.imageUrl ? detailObj.imageUrl : '~@/assets/img/avator.png'"></img>
                  <div style="width: 200px;height: 30px; lineHeight: 30px;position: absolute; bottom: 0; left: 0;textAlign: center; background: #4c484875; color: #fff;">{{detailObj.projectName}}</div>
                </div>
              </td>
            </tr>
            <tr>
              <td>不适合疾病人群</td>
              <td>{{detailObj.illLable}}</td>
            </tr>
            <tr>
              <td>描述</td>
              <td>{{detailObj.projectDesc}}</td>
            </tr>
          </table>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import echarts from 'echarts';

export default {
  props: {},
  data () {
    return {
      dialogVisible: false,
      chartBar: null,
      moodData: [],
      dataArr: [],
      detailObj: {}
    };
  },
  components: {},
  computed: {},
  methods: {
    // 查看详情
    showDetail(id) {
      // this.dialogVisible = true;
      this.detailObj = {}
      this.$http({
        url: this.$http.adornUrl(`/generator/sysprojectinfo/info/${id}`),
        method: 'get',
        params: this.$http.adornParams({})
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.detailObj = data.sysProjectInfo;
          this.dialogVisible = true;
        }
      })
    },
    initChartBar () {
      this.chartBar = echarts.init(this.$refs.echartBar)
      let option = {
        title:{
          text:"人气运动排行top5",
          textStyle:{
            fontSize:16,
            color: '#ff9f7f'
          },
          x:"35%",
          y:"2%"
        },
        color: ['#37a2da','#32c5e9','#9fe6b8','#ffdb5c','#ff9f7f','#fb7293','#e7bcf3','#8378ea'],
        tooltip: {
        },
        series : [
          {
            name:'人气值',
            type:'pie',
            // radius : [0, 130],
            itemStyle: {
              normal: {
                label: {
                  show: true,
                  position: 'inside',
                  formatter: function(params) {
                    return params.name + '(' + params.value + ')';
                  },
                }
              }
            },
            data: this.moodData.map(x => {
              return {
                value: x.clickCount,
                name: x.projectName
              }
            }),
          }
        ]
      }
      this.chartBar.setOption(option)
      window.addEventListener('resize', () => {
        this.chartBar.resize()
      })
    },
    getFilterData() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysprojectinfo/recommend`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          data.recommend.map(x => {
            x.matchingRate = (x.matchingRate * 100).toFixed(2) + '%'
          })
          this.dataArr = data.recommend
        }
      })
    },
    getMoodData() {
      this.$http({
        url: this.$http.adornUrl(`/generator/sysprojectinfo/list`),
        method: 'get',
        params: this.$http.adornParams({
          'page': 1,
          'limit': 5,
          'isClick': 1
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.moodData = data.page.list;
          this.$nextTick(() => {
            this.initChartBar()
          })
        }
      })
    },
  },
  watch: {},
  mounted() {
    // this.$nextTick(() => {
    //   this.initChartBar()
    // })
  },
  created() {
    this.getFilterData()
    this.getMoodData()
  },
}
</script>

<style scoped lang="scss">
.pe-container {
  .tips{
    color: orange;
    font-size: 18px;
    background-color: #eee;
    padding: 10px 100px;
    margin-bottom: 10px;
  }
  .pe-list{
    max-height: calc(100vh - 220px);
    overflow: auto;
  }
  .pe-item{
    display: inline-block;
    border: 2px solid #eee;
    margin-right: 5px;
    margin-bottom: 10px;
    position: relative;
    .bottom-info{
      .bottom{
        display: inline-block;
        width: 49%;
        text-align: center;
        padding: 10px 0;
        font-weight: 500;
      }
      .title{
        border-right: 2px solid #eee;
      }
      .detail{
        color: #2196f3;
        cursor: pointer;
      }
    }
  }
}

.imitate-table{
  // width: 1000px;
  table{
    border-collapse: collapse;
    width: 100%;
    tr{
      width: 100%;
      td{
        border: 1px solid #ddd;
        text-align: left;
        padding: 5px 10px;
        word-break: break-all;
        &:nth-of-type(1) {
          background-color: #eee;
          width: 300px;
          text-align:right;
        }
        &:nth-of-type(2) {
          border-left: none;
          text-align:left;
          width: calc(100vw - 300px);
        }
      }
    }
  }
}

.echart-container{
  height: calc(100vh - 220px);
  .chart-box{
    width: 100%;
    height: 100%;
  }
}
</style>