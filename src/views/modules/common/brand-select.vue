<template>
  <div>
    <el-select placeholder="请选择" v-model="brandId" filterable clearable>
      <el-option
        v-for="item in brands"
        :key="item.brandId"
        :label="item.brandName"
        :value="item.brandId"
      ></el-option>
    </el-select>
  </div>
</template>

<script>
import PubSub from 'pubsub-js'

export default {
  components: {},
  props: {},
  data () {
    // 这里存放数据
    return {
      catId: 0,
      brands: [
        {
          label: 'a',
          value: 1
        }
      ],
      brandId: '',
      subscribe: null
    }
  },
  // 计算属性类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {
    brandId (val) {
      this.PubSub.publish('brandId', val)
    }
  },
  // 方法集合
  methods: {
    getCatBrands () {
      this.$http({
        url: this.$http.adornUrl('/product/categorybrandrelation/brands/list'),
        method: 'get',
        params: this.$http.adornParams({
          catId: this.catId
        })
      }).then(({data}) => {
        this.brands = data.data
      })
    }
  },
  // 生命周期-创建完成（可以访问当前this实例）
  created () {
  },
  // 生命周期-挂载完成（可以访问DOM元素）
  mounted () {
    // 监听三级分类消息的变化
    this.subscribe = PubSub.subscribe('catPath', (msg, val) => {
      this.catId = val[val.length - 1]
      this.getCatBrands()
    })
  }
}
</script>
<style>
</style>
