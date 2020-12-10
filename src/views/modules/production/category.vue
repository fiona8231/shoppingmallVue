<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>

export default {

components: {},
data() {
return {
        menus: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
return {

};
},
//监听属性 类似于data概念
computed: {},
//监控data中的数据变化

methods:{
    getMeuns(){
        this.$http({
            url: this.$http.adornUrl('/production/category/list/tree'),
            method: 'get',
            // params: this.$http.adornParams({
            //     'page': this.pageIndex,
            //     'limit': this.pageSize,
            //     'roleName': this.dataForm.roleName
            // })
            }).then(({data})=>{
                console.log("成功获取菜单数据",data.data)
                this.menus = data.data;
            })
    },
},
watch: {},

//生命周期 - 创建完成（可以访问当前this实例）
created() {
   this.getMeuns();
},
//生命周期 - 挂载完成（可以访问DOM元素）
mounted() {

}, 

}
</script>
<style scoped>


</style>