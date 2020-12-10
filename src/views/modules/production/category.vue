<template>
  <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox
   node-key="catId">
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button v-if="node.level <=2" type="text" size="mini" @click="() => append(data)">
          Append
        </el-button>
        <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">
          Delete
        </el-button>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
    return {};
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化

  methods: {
    getMeuns() {
      this.$http({
        url: this.$http.adornUrl("/production/category/list/tree"),
        method: "get",
        // params: this.$http.adornParams({
        //     'page': this.pageIndex,
        //     'limit': this.pageSize,
        //     'roleName': this.dataForm.roleName
        // })
      }).then(({ data }) => {
        console.log("成功获取菜单数据", data.data);
        this.menus = data.data;
      });
    },
   
    append(data) {
         console.log("append",data)
      },

      remove(node, data) {
         console.log("remove", node, data)
      },
  },
  watch: {},

  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMeuns();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
};
</script>
<style scoped>
</style>