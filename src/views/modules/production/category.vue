<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      dialogVisible: false,
      expandedKey: [],
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
      console.log("append", data);
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },
    //add the append category
    addCategory() {
      console.log("提交的三级目录", this.category);
      this.$http({
        url: this.$http.adornUrl("/production/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "The menu saved successfully",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        //刷新展开
        this.getMeuns();
        //设置默认删除刷新完展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否确认删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/production/category/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          this.$message({
            message: "菜单删除成功",
            type: "success",
          });
          console.log("删除成功");
          //删除成功，刷新出新的菜单
          this.getMeuns();
          //设置默认删除刷新完展开的菜单
          this.expandedKey = [node.parent.data.catId];
        });
      });

      console.log("remove", node, data);
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