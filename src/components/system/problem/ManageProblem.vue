<template>
  <div class="main-view">
    <h2 style="text-align: center">题目列表</h2>
    <PaginateTable
        @on-page-change="onPageChange"
        @on-page-size-change="onPageSizeChange"
        v-bind:total="total"
        v-bind:page_size="page_size"
        v-bind:data="data"
        v-bind:columns="columns"
        v-bind:current="current"
        v-bind:tableLoading="tableLoading"/>
    <Button type="success" style="margin-top: 10px" @click="handleAddClick">
      <Icon type="md-add"/>
      新增
    </Button>
  </div>
</template>

<script>
import PaginateTable from "@/components/utils/PaginateTable";
import api from "@/utils/api";
import message from "@/utils/message";
import moment from "moment";
import {mapGetters} from 'vuex'

export default {
  name: "ManageProblem",
  components: {PaginateTable},
  data() {
    moment.locale('zh-cn')
    return {
      tableLoading: false,
      total: 0,
      page_size: 10,
      columns: [{
        title: '编号',
        key: 'id',
        width: 100
      }, {
        title: '标题',
        key: 'title',
        render: (h, params) => {
          return h('span', {
            style: {
              color: '#3399ff',
              cursor: 'pointer'
            },
            on: {
              click: () => {
                this.$router.push(`/system/update_problem/${params.row.id}`)

              }
            }
          }, params.row.title)
        }
      }, {
        title: '创建时间',
        render: (h, params) => {

          return h('Tooltip', {
            props: {
              transfer: true,
              content: moment(params.row.create_time).format('lll')
            }
          }, moment(params.row.create_time).fromNow())
        }
      }, {
        title: '上次修改时间',
        render: (h, params) => {
          return h('Tooltip', {
            props: {
              transfer: true,
              content: moment(params.row.last_update).format('lll')
            }
          }, moment(params.row.last_update).fromNow())
        }
      }, {
        title: '操作',
        width: 250,
        render: (h, params) => {
          return h('div', {}, [
            h('Button', {
              props: {
                type: 'info',
                size: 'small'
              }, style: {
                marginLeft: '5px'
              }, on: {
                click: () => {
                  window.open(`/problem/${params.row.id}`)
                }
              }
            }, '查看'),
            h('Button', {
              props: {
                type: 'warning',
                size: 'small'
              }, style: {
                marginLeft: '5px'
              }, on: {
                click: () => {
                  this.$router.push(`/system/update_problem/${params.row.id}`)
                }
              }
            }, '修改'),
            h('Button', {
              props: {
                type: 'error',
                size: 'small'
              },
              style: {
                marginLeft: '5px'
              },
              on: {
                click: () => {
                  this.handleProblemDelete(params.row.id)
                }
              }
            }, '删除')
          ])
        }
      }],
      data: [],
      current: 1,
      table_filters: {
        id: '',
        title: ''
      }
    }
  }, mounted() {
    this.requestTableData()
  },
  methods: {
    requestTableData(page_number) {
      this.tableLoading = true
      api.getProblemList({
        page: page_number || 1,
        page_size: this.page_size,
        id: this.table_filters.id,
        title: this.table_filters.title
      }).then(res => {
        if (res.data.err == null) {
          this.data = res.data.data.results || []
          this.total = res.data.data.count || 0
          this.tableLoading = false
          this.current = page_number
        } else {
          this.$Message.error(message.err(res.data.err));
          this.tableLoading = false
        }
      })
    },
    onPageChange(page_number) {
      this.requestTableData(page_number)
    },
    onPageSizeChange(page_size) {
      this.page_size = page_size
      this.requestTableData()
    },
    handleAddClick() {
      this.$router.push('/system/add_problem')
    },
    handleProblemDelete(id) {
      api.deleteProblemDestroy(id).then(res => {
        if (res.data.err == null) {
          this.requestTableData()
          this.$Message.success('删除成功')
        } else {
          this.$Message.error('删除失败，' + message.err(res.data.err))
        }
      })
    }
  },
  computed: {
    ...mapGetters(['web_lang'])
  }
}
</script>

<style scoped>
.main-view {
  width: 95%;
  margin-left: auto;
  margin-right: auto;
  margin-top: 10px;
}
</style>
