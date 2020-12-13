<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      draggable
      :allow-drop="allowDrop"
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
          <el-button type="text" size="mini" @click="() => modify(data)">
            Modify
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

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input
            v-model="category.name"
            autocomplete="off"
            @input="change($event)"
          ></el-input>
        </el-form-item>

        <el-form-item label="图标">
          <el-input
            v-model="category.icon"
            autocomplete="off"
            @input="change($event)"
          ></el-input>
        </el-form-item>

        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
            @input="change($event)"
          ></el-input>
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
  components: {},
  data() {
    return {
   
      maxLevel: 0,
      dialogType: "",
      icon: "",
      productUnit: "",
      title: "",

      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
      },
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
    change(e) {
      this.$forceUpdate();
    },

    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop:", draggingNode, dropNode, type);
      
      this.countNodeLevel(draggingNode.data);
      let deep = this.maxLevel - draggingNode.data.catLevel + 1;

      if (type == "inner") {
        return (deep + dropNode.level) <= 3;
      } else {
        return (deep + dropNode.parent.level) <= 3;
      }
     
    },
    countNodeLevel(node) {
      //求最大深度
      if (node.children != null && node.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel;
          }
          this.countNodeLevel(node.children[i]);
        }
      }
    },

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
      this.dialogType = "add";
      this.title = "Add Category";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = data.data.catId;
      this.category.icon = data.data.icon;
      this.category.productUnit = data.data.icon;
      this.category.parentCid = data.data.parentCid;
      this.category.catLevel = data.data.catLevel;
      this.category.showStatus = 1;
      this.category.sort = 0;
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "modify") {
        this.modifyCategory();
      }
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
    modify(data) {
      console.log("要修改的数据", data);
      this.dialogType = "modify";
      this.title = "Modidy Category";
      this.dialogVisible = true;
      //发送请求获得当前节点最新数据
      this.$http({
        url: this.$http.adornUrl(`/production/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功
        console.log("要回显的数据", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.icon;
        this.category.parentCid = data.data.parentCid;
        this.category.catLevel = data.data.catLevel;
        this.category.showStatus = data.data.showStatus;
        this.category.sort = data.data.sort;
      });
    },

    //modify categofry
    modifyCategory() {
      // var {catId, name, icon, productUnit} = this.category;

      this.$http({
        url: this.$http.adornUrl("/production/category/update"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });
        this.dialogVisible = false;
        //删除成功，刷新出新的菜单
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