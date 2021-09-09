<template>
  <el-table-column :prop='column.prop' :width='column.width' :min-width='column.minWidth'
                   :fixed='column.fixed' :align='column.align' :sortable='column.sortable'
                   :label='column.label' :show-overflow-tooltip='column.showTooltip'>
    <!-- 列头 -->
    <template slot="header" slot-scope="scope">
      <span v-if='!column.labelRender&&!column.labelSlot'>{{column.label}}</span>
      <!-- 表头插槽内容 -->
      <template v-else-if="column.labelSlot">
        <slot :name='column.labelSlot' :column="column"></slot>
      </template>
      <expand-dom v-else :column="column" :render="column.labelRender"></expand-dom>
    </template>
    <template slot-scope="scope">
      <!--  默认渲染内容 -->
      <template v-if="!column.render&&!column.slot">
        <!-- 格式化内容 -->
        <span v-if="column.formatter" v-html="column.formatter(scope.row, column)"></span>
        <span v-else>{{scope.row[column.prop]}}</span>
      </template>
      <!-- 插槽内容 -->
      <template v-else-if="column.slot">
        <slot :name='column.slot' :index='scope.$index' :row="scope.row" :column='column'></slot>
      </template>
      <!-- 自定义渲染内容 -->
      <expand-dom v-else :column="column" :row="scope.row" :render="column.render" :index="index">
      </expand-dom>
    </template>
    <!-- 多级表头 -->
    <template v-if="column.children">
      <tableColumn v-for="(item,num) in column.children" :key='item.prop' :column='item'
                   :index='num'></tableColumn>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'tableColumn',
  components: {
    // https://cn.vuejs.org/v2/guide/render-function.html 函数式组件
    expandDom: {
      functional: true,
      props: {
        row: Object,
        render: Function,
        index: Number,
        column: {
          type: Object,
          default: null
        }
      },
      render: (createElement, context) => {
        const params = {
          row: context.props.row,
          index: context.props.index
        }
        if (context.props.column) params.column = context.props.column
        return context.props.render(createElement, params)
      }
    }
  },
  props: {
    index: Number,
    column: {
      type: Object
    }
  }
}
</script>

<style lang="scss" scoped>
</style>