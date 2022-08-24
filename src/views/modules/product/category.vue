<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="拖拽开启"
      inactive-text="拖拽关闭">
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button @click="batchDelete" type="danger">批量删除</el-button>
    <el-tree :data="this.menu"
             show-checkbox
             :default-expanded-keys="this.expandedKeys"
             node-key="catId"
             :props="defaultProps" @node-click="handleNodeClick"
             :expand-on-click-node="false"
             :allow-drop="allowDrop"
             @node-drop="handleDrop"
             :draggable="draggable"
             ref="menuTree"
    >
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!--只有父级目录显示添加按钮-->
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">edit</el-button>
          <!--当该节点没有子目录才显示删除按钮-->
          <el-button v-if="node.childNodes.length === 0" type="text" size="mini" @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category" ref="category">
        <el-form-item label="菜单名称" prop="name">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" prop="productUnit">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      pCid: [],
      draggable: false,
      updateNodes: [],
      title: '',
      maxLevel: 0,
      dialogVisible: false,
      menu: [],
      category: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productCount: 0,
        icon: '',
        productUnit: ''
      },
      expandedKeys: [],
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
    batchDelete () {
      let catIds = []
      let names = []
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId)
        names.push(checkedNodes[i].name)
      }
      this.$confirm(`确定进行批量删除【${names}】操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '菜单批量删除成功',
              type: 'success',
              duration: 500,
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {
      })
    },
    batchSave () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '菜单顺序修改成功',
            type: 'success',
            duration: 500,
            onClose: () => {
              this.getDataList()
              this.expandedKeys = this.pCid
              this.maxLevel = 0
              this.updateNodes = []
              // this.pCid = 0
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    // 拖拽完成后做的事
    handleDrop (draggingNode, dropNode, dropType, ev) {
      console.log('handleDrop: ', draggingNode, dropNode, dropType, ev)
      // 当完成拖拽功能后 需要 修改三个值 1 parent_cid 2 cat_level 3 sort
      // 1 获取拖拽后节点的父id
      // eslint-disable-next-line no-unused-vars
      let pCid = 0
      let siblings = null
      if (dropType === 'before' || dropType === 'after') {
        // 如果是前后拖动的话 那么该节点的父id 就是兄弟节点的父id
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        // 如果是插入关系则 父Id为
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)
      // 2、当前拖拽节点的最新顺序，
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level
          if (draggingNode.level !== siblings[i].level) {
            catLevel = siblings[i].level
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel
          })
        } else {
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i
          })
        }
        // 如果遍历到被拖拽的节点，需要修改它的子节点id和层级
      }
      console.log('updateNodes', this.updateNodes)
    },
    // 修改子节点层级
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data
          this.updateNodes.push({catId: cNode.catId, catLevel: node.childNodes[i].level})
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    allowDrop (draggingNode, dropNode, type) {
      console.log('allowDrop', draggingNode, dropNode, type)
      // 先计算拖拽节点的最大深度
      this.countNodeLevel(draggingNode)
      let deep = Math.abs(this.maxLevel - draggingNode.data.catLevel) + 1
      console.log('deep', deep)
      if (type === 'inner') {
        return (deep + dropNode.level) <= 3
      } else {
        return (deep + dropNode.parent.level) <= 3
      }
    },
    countNodeLevel (node) {
      this.maxLevel = node.catLevel
      if (node.childNodes !== null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    submitData () {
      if (this.category.catId) {
        this.updateCategory()
      } else {
        this.addCategory()
      }
      this.dialogVisible = false
    },
    edit (data) {
      console.log('edit', data)
      this.title = '修改菜单'
      this.dialogVisible = true
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        console.log(data)
        this.category.catId = data.data.catId
        this.category.name = data.data.name
        this.category.parentCid = data.data.parentCid
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.category.catLevel = data.data.catLevel
        this.category.showStatus = data.data.showStatus
        this.category.productCount = data.data.productCount
        this.category.sort = data.data.sort
      })
    },
    addCategory () {
      console.log('addCategory', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '菜单添加成功',
            type: 'success',
            duration: 500,
            onClose: () => {
              this.getDataList()
              this.expandedKeys = [this.category.parentCid]
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    updateCategory () {
      console.log('updateCategory', this.category)
      const {catId, icon, productUnit, name} = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, icon, productUnit, name}, false)
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '菜单修改成功',
            type: 'success',
            duration: 500,
            onClose: () => {
              this.getDataList()
              this.expandedKeys = [this.category.parentCid]
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    append (data) {
      console.log('apend', data)
      this.title = '新增菜单'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.showStatus = 1
      this.category.productCount = 0
      this.category.sort = 0
    },
    remove (node, data) {
      console.log('remove', node, data)
      var ids = [data.catId]
      console.log(this.expandedKeys)
      this.$confirm(`确定对【${data.name}】进行删除操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '删除成功',
              type: 'success',
              duration: 500,
              onClose: () => {
                this.getDataList()
                this.expandedKeys = [node.parent.data.catId]
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      }).catch(() => {
      })
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
