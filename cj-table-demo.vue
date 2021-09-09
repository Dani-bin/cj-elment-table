<template>
  <div class="cj-page-wrap">
    <cj-table :searchData='searchData' url='/bgs/queryData' :columnList='columnList' :init='false'
              :selectable='selectable' :summary-method="getSummaries" :show-summary='true'
              ref='cjTabel'>
      <template slot='label' slot-scope="scope">
        <span @click="getColumn(scope)">插槽表头</span>
      </template>
      <template slot='slotDemo' slot-scope="scope">
        <span>这是插槽内容...</span>
      </template>
      <template slot='operation' slot-scope="scope">
        <cj-icon-btn icon="svg-btn-edit" type="primary" tipText="编辑" @click="editRow(scope)">
        </cj-icon-btn>
      </template>
    </cj-table>
  </div>
</template>

<script>
import cjTabel from './index'
export default {
  components: {
    cjTabel,
  },
  data() {
    return {
      tabelText: '',
      searchData: {},
      columnList: [
        {
          prop: 'date',
          width: '140',
          sortable: true,
          label: '日期',
          formatter: (row, column, cellValue, index) => {
            return row.date
          }
        },
        {
          prop: 'name',
          label: '姓名'
        },
        {
          prop: 'age',
          label: '年龄',
          labelRender: (h, params) => {
            return h('el-input', {
              attrs: {
                placeholder: '请输入'
              },
              props: {
                size: 'mini',
                value: this.tabelText
              }, // 组件的props
              on: {
                input: (value) => {
                  this.tabelText = value
                }
              }
            })
          }
        },
        {
          prop: 'sex',
          label: '性别',
          render: (h, params) => {
            return h('el-tag', {
              props: {
                type: params.row.sex === 'boy' ? 'success' : 'danger'
              }, // 组件的props
              on: {
                click: () => {
                  this.clickHandler(params)
                }
              }
            }, params.row.sex === 'boy' ? '男' : '女')
          }
        },
        {
          label: '插槽',
          labelSlot: 'label',
          slot: 'slotDemo'
        },
        {
          prop: 'addressInfo',
          label: '地址信息',
          children: [
            {
              prop: 'province',
              label: '省份',
              width: 140
            },
            {
              prop: 'city',
              label: '市区',
              width: 140
            },
            {
              prop: 'address',
              label: '地址',
              width: 140
            }
          ]
        },
        {
          label: '操作',
          slot: 'operation'
        }
      ]
    }
  },
  mounted() {
    // this.$refs.cjTabel.search() //手动获取数据
    this.$refs.cjTabel.tableData = [{
      date: '2016-05-03',
      age: 10,
      sex: 'boy',
      name: '王小虎',
      province: '上海',
      city: '普陀区',
      address: '上海市普陀区金沙江路 1518 弄',
      zip: 200333
    }, {
      date: '2016-05-02',
      age: 10,
      sex: 'boy',
      name: '王小虎',
      province: '上海',
      city: '普陀区',
      address: '上海市普陀区金沙江路 1518 弄',
      zip: 200333
    }, {
      date: '2016-05-04',
      age: 10,
      sex: 'boy',
      name: '王小虎',
      province: '上海',
      city: '普陀区',
      address: '上海市普陀区金沙江路 1518 弄',
      zip: 200333
    }]
  },
  methods: {
    getData(data, resolve) {
      return this.$axios.post('/api/jskq/kqz/common/getJzgPageList', {
        ...data,
        search: JSON.stringify({
          kqzId: ''
        })
      }).then(res => {
        resolve(res.data.rows, res.data.total)
      })
    },
    getColumn(scope) {
      console.log(scope)
    },
    selectable(row) {
      return true
    },
    clickHandler(row) {
      console.log(row)
    },
    getSummaries(param) {
      const { columns, data } = param
      const sums = []
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '总价'
          return
        }
        const values = data.map(item => Number(item[column.property]))
        if (!values.every(value => isNaN(value))) {
          sums[index] = values.reduce((prev, curr) => {
            const value = Number(curr)
            if (!isNaN(value)) {
              return prev + curr
            } else {
              return prev
            }
          }, 0)
          sums[index] += ' 元'
        } else {
          sums[index] = 'N/A'
        }
      })
      return sums
    },
    editRow(row) {
      console.log(row)
      console.log(this.tabelText)
      this.tabelText = 'aaaaaaaaa'
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
