<template>
  <Layout>
    <Content>
      <Breadcrumb>
        <BreadcrumbItem>预警管理系统</BreadcrumbItem>
        <BreadcrumbItem>预警处理结果台账</BreadcrumbItem>
      </Breadcrumb>
      <Card>
        <searchCondition ref="condition" @searchClick="preSearch" :excepts="['status']" />
        <hy-table
          highlight-row
          border
          :current="pageInfo.pageNo"
          :columns="tableList.columns"
          :data="tableList.data"
          :total="Number(pageInfo.total)"
          :pageSize="Number(pageInfo.pageSize)"
          pageType="small"
          show-elevator
          show-sizer
          class="refer-apply"
          @on-page-change="pageChange" />
        <Modal
          v-model="showModal"
          title="预警信息详情"
          :width="960">
          <detail ref="detail" key="deal" />
          <div slot="footer">
            <Button class="modalBtn" type="primary" @click="showModal = false" size="large">确定</Button>
          </div>
        </Modal>
      </Card>
    </Content>
  </Layout>
</template>

<script>
import {mapState} from 'vuex'
import api from '@/api/axiosApi'
import warnApiList from '@/api/warnApiList'
import searchCondition from './searchCondition.vue'
import detail from './detail.vue'
import {levels, allStatus} from './constant'

function defaultDisplay(h, params, key) {
  return h('span', params.row[key] || '--')
}

export default {
  components: {
    searchCondition,
    detail
  },
  data () {
    const vm = this
    return {
      showModal: false,
      tableList: {
        columns: [
          {
            type: 'index',
            title: '序号',
            width: 60,
            align: 'center'
          },
          {
            title: '预警级别',
            key: 'level',
            render: (h, params) => {
              const {level} = params.row
              let text = '--'
              for (let l of levels) {
                if (l.value == level) {
                  text = l.label
                  break
                }
              }
              return h('span', text)
            }
          },
          {
            title: '预警类型',
            key: 'typeName',
            render: (h, params) => {
              return defaultDisplay(h, params, 'typeName')
            }
          },
          {
            title: '预警标题',
            key: 'title',
            render: (h, params) => {
              return defaultDisplay(h, params, 'title')
            }
          },
          {
            title: '应用程序名称',
            key: 'appName',
            render: (h, params) => {
              return defaultDisplay(h, params, 'appName')
            }
          },
          {
            title: '责任人',
            key: 'dutyName',
            render: (h, params) => {
              return defaultDisplay(h, params, 'dutyName')
            }
          },
          {
            title: '责任人处理信息',
            key: 'remarks',
            render: (h, params) => {
              return defaultDisplay(h, params, 'remarks')
            }
          },
          {
            title: '预警确认时长（h）',
            key: 'contimeL',
            render: (h, params) => {
              return defaultDisplay(h, params, 'contimeL')
            }
          },
          {
            title: '预警处理时长（h）',
            key: 'hantimeL',
            render: (h, params) => {
              return defaultDisplay(h, params, 'hantimeL')
            }
          },
          {
            title: '预警时间',
            key: 'warntime',
            render: (h, params) => {
              return defaultDisplay(h, params, 'warntime')
            }
          },
          {
            title: '操作',
            key: 'address',
            align: 'center',
            render: (h, params) => {
              const vm = this
              const {
                id
              } = params.row
              const style = {
                marginRight: '5px'
              }
              // 详情
              const info = h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                style,
                domProps: {
                  innerHTML: '详情'
                },
                on: {
                  click () {
                    vm.showModal = true
                    vm.$refs.detail.refresh(id)
                  }
                }
              })
              const btns = []
              btns.push(info)
              return h('span', btns)
            }
          }
        ],
        data: []
      },
      pageInfo: {
        pageNo: 1,
        pageSize: 10,
        total: 0
      }
    }
  },
  mounted () {
    this.search()
  },
  methods: {
    preSearch() {
      this.pageInfo.pageNo = 1
      this.search()
    },
    pageChange(pageNo, pageSize) {
      this.pageInfo.pageNo = pageNo
      this.pageInfo.pageSize = pageSize
      this.search()
    },
    // 获取所有处理台账的列表
    search () {
      const vm = this
      const data = vm.$refs.condition.getCondition()
      api(warnApiList.findWarnResultList, {
        data,
        ...vm.pageInfo
      }).then(res => {
        if (res.data.errcode === 0) {
          const result = res.data.data
          vm.pageInfo.pageNo = result.pageNum
          vm.pageInfo.total = result.total
          vm.tableList.data = result.list
        }
      }, error => {console.log(error)})
    }
  }
}
</script>

<style lang='less'>
.refer-apply{
  .handle{
    margin: 0 5px;
    display: inline-block;
    cursor: pointer;
    color: #2d8cf0;
  }
}
#applyModal .ivu-form-item-content{
  height: auto !important;
}
html .ivu-layout-content, body .ivu-layout-content, #app .ivu-layout-content, .ivu-layout .ivu-layout-content {
  padding-bottom: 100px;
}
</style>
