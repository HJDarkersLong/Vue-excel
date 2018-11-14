# Vue-excel
使用vue 来创建excel文件
基于vue创建的一个生成excel的方法


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
      }
      
      
      
      
      methods 里面有这个方法，是点击下载执行的方法，该方法首先是访问内部接口，查询出需要展示的数据，用数组exportList进行接收（如果不需要数据例如，需求只需要下载Excel模板，不需要数据，则不用进行接口访问，但是）exportList需要定义，设置为空数组即可
      然后执行 require.ensure 方法，设置表头，列名，文件名等相关设置formatJson方法如下：（注意的是filterVal和filterValSecond相等的时候会直接填入excel，如果查询出来的是对象需要展示的是内部属性，则需要向如下方法一样输出即可）
      
      
      
      
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
      }
      
      介绍完毕
