<template>
  <div v-loading='tableloading' class="cj-table">
    <slot v-if='historySelect' name='statistics'>
      <el-alert :title="`已选择 ${tableSelectRow.length} 条数据`" :closable="false" type="success"
                show-icon>
      </el-alert>
    </slot>

    <el-table :data='tableData' stripe border style='width: 100%' @sort-change='sortChange'
              :row-key='rowKey' :span-method='spanMethod' :summary-method='summaryMethod'
              :show-summary='showSummary' :height='height' :max-height='maxHeight'
              :header-cell-class-name='getCellClass' @select='cellSelect' @select-all='selectAll'
              @cell-click='tableCellClick' @selection-change='tableSelectionChange'
              highlight-current-row ref='mainTable' class="main-table">
      <el-table-column v-if='selection' fixed type='selection' :selectable='selectable' width='55'
                       :reserve-selection='historySelect' align="center">
      </el-table-column>
      <el-table-column v-if='showIndex' fixed label="序号" align="center" width='65'
                       show-overflow-tooltip>
        <template slot-scope="scope">
          {{scope.$index + (pageSize * (currentPage - 1)) + 1}}
        </template>
      </el-table-column>
      <template v-for="(item,index) in columnList">
        <tableColumn :key='item.prop' :column='item' :index='index'>
          <template v-if='item.labelSlot' :slot="item.labelSlot" slot-scope="scope">
            <slot :name='item.labelSlot' :column='scope.column'></slot>
          </template>
          <template v-if='item.slot' :slot="item.slot" slot-scope="scope">
            <slot :name='item.slot' :index='scope.index' :row="scope.row"></slot>
          </template>
        </tableColumn>
      </template>
    </el-table>
    <el-pagination @size-change='handleSizeChange' @current-change='handleCurrentChange'
                   v-if='pageFlag' :current-page='currentPage' :page-sizes='pageSizes'
                   :pager-count='pagerCount' :page-size='pageSize'
                   layout='total, sizes, prev, pager, next, jumper' :total='total'>
    </el-pagination>
  </div>
</template>

<script>
import tableColumn from './tableColumn'
export default {
  name: 'CjTable',
  components: {
    tableColumn
  },
  data() {
    return {
      sortField: '',
      order: '',
      tableloading: false,
      tableSelectRow: [], // 勾选数据
      tableData: [],
      currentPage: 1,
      pageSize: 10,
      total: 0
    }
  },
  props: {
    height: [Number, String],
    maxHeight: [Number, String],
    // 是否立即执行 search查询数据
    init: {
      type: Boolean,
      default: true
    },
    // 是否记录历史选择的数据  跨分页选择（必传rowKey）
    historySelect: {
      type: Boolean,
      default: false
    },
    //  历史选择的数据
    hisSelectData: {
      type: Array,
      default: () => []
    },
    // 表格是否多选
    multiple: {
      type: Boolean,
      default: true
    },
    // 表格数据行点击是否勾选
    rowClickSelected: {
      type: Boolean,
      default: false
    },
    // 是否在表尾显示合计行
    showSummary: {
      type: Boolean,
      default: false
    },
    // 行数据的 Key 记录历史选择数据时，必传
    rowKey: String,
    // 仅对 type=selection 的列有效，类型为 Function，Function 的返回值用来决定这一行的 CheckBox 是否可以勾选
    selectable: Function,
    // 自定义获取数据方法
    getData: Function,
    // 自定义的合计计算方法
    summaryMethod: Function,
    // 合并行或列的计算方法
    spanMethod: Function,
    // 请求的参数
    searchData: {
      type: [Object, String]
    },
    // 是否分页
    pageFlag: {
      type: Boolean,
      default: true
    },
    // 列表数据请求地址
    url: {
      type: String,
      default: ''
    },
    /*
    列 column格式：
    {
      width:'', // 列宽 string
      minWidth:'', // 最小宽
      fixed:'left',  // 是否固定 true, left, right
      align: 'left',  // 对齐方式
      prop:'',  // 列prop string
      label:'', // 列名 string
      sortable: false, // 是否排序 boolean/string   true, false, 'custom'
      showTooltip: false,  // 默认当内容过长被隐藏时不显示tooltip   boolean
      formatter:(row, column)=>{},  // 内容格式化函数
      labelRender:(row, column)=>{},  // 表头render函数
      render:(row, column)=>{}  // 内容render函数
      labelSlot: '' // 表头插槽名称
      slot: '' // 内容插槽名称
    }
    **/
    columnList: {
      type: Array,
      default: () => []
    },
    // 表格多选框配置
    selection: {
      type: Boolean,
      default: false
    },
    // 表格序列号配置
    showIndex: {
      type: Boolean,
      default: false
    },
    /*
      JyPagination 组件props
    **/
    pagerCount: Number,
    // 自定义分页条数
    pageSizes: Array
  },
  computed: {
    hisSelectId() {
      return this.rowKey ? this.hisSelectData.map(val => val[this.rowKey]) : []
    }
  },
  created() {
    this.init && this.search()
  },
  methods: {
    // 是否隐藏全选
    getCellClass({ columnIndex }) {
      if (columnIndex === 0 && !this.multiple) {
        return 'hidden-cell'
      }
    },
    // 查询 true查询第一页数据
    search(flag) {
      if (flag) this.currentPage = 1
      // prop传递 异步
      this.$nextTick(() => {
        this.getTableData()
      })
    },
    // 重置 页码 排序条件
    reset() {
      this.currentPage = 1
      this.sortField = ''
      this.order = ''
      this.$refs.mainTable.columns.forEach(val => {
        val.order = ''
      })
      // prop传递 异步
      this.$nextTick(() => {
        this.getTableData()
      })
    },
    // 排序
    sortChange({ column, prop, order }) {
      this.sortField = prop
      this.order = order === 'ascending' ? 'asc' : order === 'descending' ? 'desc' : ''
      this.search()
    },
    // 用户手动勾选数据行的 Checkbox 时触发
    cellSelect(selection, row) {
      this.$emit('cellSelect', selection, row)
    },
    // 数据行点击
    tableCellClick(row, column, cell, event) {
      // 单击勾选数据行
      this.rowClickSelected && this.$refs.mainTable.toggleRowSelection(row)
      this.$emit('tableCellClick', row, column, cell, event)
    },
    // 手动勾选全选 Checkbox
    selectAll(selection) {

    },
    // 表格选择
    tableSelectionChange(selection) {
      // 如果是单选
      if (!this.multiple && selection.length > 1) {
        this.$refs.mainTable.clearSelection()
        const row = selection.pop()
        selection = [row]
        this.$refs.mainTable.toggleRowSelection(row, true)
      }
      this.tableSelectRow = selection
      this.$emit('tableSelectionChange', {
        tableSelectRow: this.tableSelectRow
      })
    },
    // 分页SIZE
    handleSizeChange(val) {
      this.pageSize = val
      this.search()
    },
    // 页码改变
    handleCurrentChange(val) {
      this.currentPage = val
      this.getTableData()
    },
    // 回显已选择的数据——勾选
    setHisSelect() {
      // console.log('已选择的数据：', this.hisSelectId)
      this.$nextTick(() => {
        this.tableData.forEach(val => {
          this.$refs.mainTable.toggleRowSelection(val, this.hisSelectId.includes(val[this.rowKey]))
        })
      })
    },
    // 获取数据
    getTableData() {
      this.tableloading = true
      // 自定义获取数据
      if (this.getData) {
        this.getData({
          currPage: this.currentPage - 1,
          limit: this.pageSize,
          order: this.order,
          sortField: this.sortField,
          page: this.pageFlag
        }, (tableData, total) => {
          this.tableloading = false
          this.tableData = tableData
          this.total = total
          // 回显历史选择数据勾选状态
          this.setHisSelect()
          if (this.currentPage > 1 && !tableData.length) {
            this.currentPage--
            this.getTableData()
          }
        })
      } else {
        this.$axios({
          url: this.url,
          method: 'post',
          data: {
            currPage: this.currentPage - 1,
            limit: this.pageSize,
            order: this.order,
            sortField: this.sortField,
            search: this.searchData,
            page: this.pageFlag
          }
        }).then(res => {
          if (res.result) {
            this.total = res.data.total
            this.tableData = res.data.rows
            // 回显历史选择数据勾选状态
            this.setHisSelect()
            if (this.currentPage > 1 && !res.data.rows.length) {
              this.currentPage--
              this.getTableData()
            }
          } else {
            this.$message.error(res.msg)
          }
        }).finally(() => {
          this.tableloading = false
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.main-table {
  /deep/ .hidden-cell {
    .cell {
      display: none;
    }
  }
}
</style>