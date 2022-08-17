<template>
  <el-tree :data="this.menu" :props="defaultProps" @node-click="handleNodeClick"
           :expand-on-click-node="false" show-checkbox node-key:="catId">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!--只有父级目录显示添加按钮-->
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <!--当该节点没有子目录才显示删除按钮-->
          <el-button v-if="node.childNodes.length === 0" type="text" size="mini" @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
  </el-tree>
</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      menu: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  created () {
    this.getDataList()
  },
  methods: {
    append (data) {
      console.log(data)
    },
    remove (node, data) {
      console.log('node:', node, ' data:', data)
    },
    handleNodeClick (data) {
    },
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.menu = data.data
        } else {
          this.menu = []
        }
        this.dataListLoading = false
      })
    }
  }
}
</script>

<style scoped>

</style>
