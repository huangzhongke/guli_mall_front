<template>
  <el-tree :data="this.menu"
           node-key="catId"
           ref="menuTree"
           :props="defaultProps"
           @node-click="nodeClick"
  >
  </el-tree>
</template>

<script>
export default {
  name: 'categoryTree',
  created () {
    this.getDataList()
  },
  data () {
    return {
      menu: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    nodeClick (data, node, component) {
      this.$emit('tree-node-click', data, node, component)
    },
    getDataList () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.menu = data.data
        } else {
          this.menu = []
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
