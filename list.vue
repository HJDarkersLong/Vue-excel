<template>
  <section class="content">
    <div class="box box-primary">
      <div class="box-body">
        <form class="form-horizontal">
          <div class="row row-margin-bottom">
            <div class="col-sm-2">
              <input class="form-control input-sm" type="text" v-model="param.id" placeholder="订单号"/>
            </div>
            <div class="col-sm-2">
              <input class="form-control input-sm" type="text" v-model="param.machineName" placeholder="机柜名"/>
            </div>
            <div class="col-sm-2">
              <input class="form-control input-sm" type="text" v-model="param.mobile" placeholder="手机号"/>
            </div>
            <div class="col-sm-2">
              <select class="form-control input-sm" v-model="param.status" name="" >
                <option value="">状态</option>
                <option value="0">未定损</option>
                <option value="1">已定损待确认</option>
                <option value="2">有异议</option>
                <option value="3">已判定</option>
                <option value="4">已支付</option>
              </select>
            </div>
            <div class="col-sm-2">
              <DatePicker v-model="param.beginTime"  v-validate="'required'"
                          data-vv-value-path="param.createTime" placeholder="开始时间"></DatePicker>
            </div>
            <div class="col-sm-2">
              <DatePicker v-model="param.endTime"  v-validate="'required'"
                          data-vv-value-path="param.createTime" placeholder="结束时间"></DatePicker>
            </div>
            <!--<div class="col-sm-2">-->
            <!--<TimePicker v-model="param.beginTime"  v-validate="'required'"-->
            <!--data-vv-value-path="param.createTime" data-vv-name="开始时间"></TimePicker>-->
            <!--</div>-->
            <!--<div class="col-sm-2">-->
            <!--<TimePicker v-model="param.endTime"  v-validate="'required'"-->
            <!--data-vv-value-path="param.createTime" data-vv-name="结束时间"></TimePicker>-->
            <!--</div>-->
            <div class="col-sm-2">
              <select class="form-control input-sm" v-model="param.channel" name="" >
                <option value="">支付类型</option>
                <option value="1">会员卡</option>
                <option value="2">微信</option>
                <option value="3">支付宝</option>
              </select>
            </div>
            <div class="col-sm-2">
              <a class="btn btn-primary btn-sm" @click="handleDownload" >导出</a>
              <!--<router-link to="/order/rentorder/add" class="btn btn-primary btn-sm">添加</router-link>-->
            </div>
          </div>
        </form>
        <!--Query End-->

        <div class="row" style="width:100%;overflow-x:auto">
          <div class="col-md-12">
            <table class="table table-bordered table-hover">
              <thead>
              <tr>
                <!--<th>序号</th>-->
                <th>订单号</th>
                <th>玩具柜</th>
                <th>用户昵称</th>
                <th>会员手机</th>
                <th>玩具名</th>
                <th>货号</th>
                <th>初次定损</th>
                <th>定损人</th>
                <th>二次定损</th>
                <th>理赔金额</th>
                <th>状态</th>
                <th>定损照片</th>
                <th>客户异议反馈</th>
                <th>核定理赔金额</th>
                <th>支付类型</th>
                <th>租赁时间</th>
                <th>归还时间</th>
                <th>定损</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="(item,index) of page.list" :key="item.id">
                <!--<td>{{index+1}}</td>-->
                <td>{{item.id}}</td>
                <td><span v-if="item.machine">{{item.machine.name}}</span></td>
                <td><span v-if="item.member">{{item.member.name}}</span></td>
                <td><span v-if="item.member">{{item.member.mobile}}</span></td>
                <td><span v-if="item.toy">{{item.toy.name}}</span></td>
                <td><span v-if="item.toy">{{item.toy.id}}</span></td>
                <td>
                  <span v-if="item.firstRule">{{item.firstRule.name}}</span>
                </td>
                <td><span v-if="item.finalJudgeMan">{{ item.finalJudgeMan.name }}</span></td>
                <td>
                  <span v-if="item.realRule">{{item.realRule.name}}</span>
                </td>
                <td>{{ (item.claim *0.01).toFixed(2) }}</td>
                <td><span v-if="item.status">{{getStatusDesc(item.status)}}</span>
                </td>
                <td>
                  <a @click="onSheft(item.iconList)">查看</a>
                  <pictures :config="pictureModalConfig" v-model="obj.pictures" ref="pictures" ></pictures>
                  <!--<OnePicture :editable="false" :fileId="item.icon"></OnePicture>-->
                </td>
                <td>{{ item.customFeed }}</td>
                <td>{{ (item.realClaim * 0.01).toFixed(2) }}</td>
                <td><span v-if="item.channel == 1">会员卡</span>
                  <span v-else-if="item.channel == 2">微信</span>
                  <span v-else-if="item.channel == 3">支付宝</span>
                </td>
                <td><span v-if="item.rentOrder">{{ item.rentOrder.createTime }}</span></td>
                <td><span >{{ item.createTime }}</span></td>
                <td>
                  <router-link  v-if="item.status == 0 || item.status == 2" :to='"/order/damag/form/" + item.id '>定损</router-link>
                  <router-link  v-if="item.status == 1 || item.status == 3" :to='"/order/damag/form/" + item.id '>重新定损</router-link>
                </td>
              </tr>
              </tbody>
            </table>
            <Pagination :page="page" @page="handlerPage(arguments)"></Pagination>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
  import PageMixin from '@mixins/PageMixin'
  import OnePicture from '@components/form/file-upload/picture/OnePicture'
  import PictureModal from './PictureModal.vue'

  export default {
    mixins: [PageMixin],
    components: {
      OnePicture,
      pictures: PictureModal
    },
    data: function () {
      return {
        actions: {
          list: {method: 'get', url: '/admin/order/toycheckorder/list'}
        },
        param: {
          mobile: '',
          channel: '',
          status: '',
          type: '',
          description: ''
        },
        obj: {
          claim: '',
          toy: {},
          machine: {},
          machineCell: {},
          member: {},
          status: '',
          channel: '',
          pictures: {},
          firstRule: {},
          realRule: {},
          finalJudgeMan: {},
          rentOrder: {}
        },
        pictureModalConfig: {
          show: false,
          title: '定损图片',
          iconList: ''
        }
      }
    },
    methods: {
      onSheft (iconList) {
        this.pictureModalConfig.iconList = iconList
        this.pictureModalConfig.show = true
      },
      handleDownload () {
        if (this.param.beginTime == null && this.param.endTime == null ) {
          alert('请选择时间段后导出！')
          return
        }
        this.$api.http.post('/admin/order/getExportExcel/exportDamageOrderList', this.param).then((response) => {
          let result = response.data
          if (result.isOk) {
            this.exportList = result.data
            console.log(result)
          } else {
            this.doAlert(result)
          }
        }).then((response) => {
          require.ensure([], () => {
            const { export_json_to_excel } = require('../exportForm/Export2Excel')
            const tHeader = [    '订单号','玩具柜',      '用户昵称',    '会员手机',      '玩具名',   '货号',  '初次定损',        '定损人',            '二次定损',   '理赔金额',  '状态',   '客户异议反馈',  '核定理赔金额', '支付类型', '租赁时间',      '归还时间']
            const filterVal =       ['id','machine',    'member',      'member',        'toy',     'toy' ,  'firstRule',      'finalJudgeMan',     'realRule',   'claim',    'status', 'customFeed',   'realClaim' ,   'payList', 'rentOrder',     'createTime']
            const filterValSecond = ['id','machineName','memberName',  'memberMobile',  'toyName', 'toyId', 'firstRuleDesc',  'finalJudgeManName', 'realRuleName','CLAIM',   'statusDesc', 'customFeed',   'REALCLAIM' ,   'payList', 'rentOrderTime', 'createTime']
            const list = this.exportList
            console.log(list)
            const data = this.formatJson(filterVal, filterValSecond, list)
            export_json_to_excel(tHeader, data, '定损订单')
          })
        })
      },
      formatJson (filterVal, filterValSecond, jsonData) {
        let _this = this
        const jsonDataTemp = jsonData.map(function (v) {
          return (filterVal).map(function (j, s) {
            console.log(filterValSecond[s])
            console.log(j)
            if (j === filterValSecond[s]) {
              return v[j]
            } else {
              switch (filterValSecond[s]) {
                case 'memberName': return v[j] == null ? '' : v[j].name
                case 'machineName':return v[j] == null ? '' : v[j].name
                case 'memberMobile':return v[j] == null ? '' : v[j].mobile
                case 'toyName':return v[j] == null ? '' : v[j].name
                case 'toyId':return v[j] == null ? '' : v[j].id
                case 'firstRuleDesc':return v[j] == null ? '' : v[j].name
                case 'finalJudgeManName':return v[j] == null ? '' : v[j].name
                case 'realRuleName':return v[j] == null ? '' : v[j].name
                case 'rentOrderTime':return v[j].createTime == null ? '' : v[j].createTime
                case 'payDesc':return v[j][0] == null ? '' : v[j][0].desc
                case 'REALCLAIM':return v[j] * 0.01
                case 'CLAIM':return v[j] * 0.01
                case 'statusDesc':return _this.getStatusDesc(v[j])
              }
            }
          })
        })
        return jsonDataTemp
      },
      getStatusDesc (status) {
        switch (status) {
          case '0' :return '未定损'
          case '1' :return '已定损'
          case '2' :return '有异议'
          case '3' :return '已判定'
          case '4' :return '已支付'
        }
      }
    }
  }
</script>

