<template>
  <div>
    <el-tree
      ref="menuTree"
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      @node-click="nodeClick"
    >
    </el-tree>
  </div>
</template>

<script>
export default {
  props: {},
  methods: {
    // 获取数据列表
    getMeus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log(data.data);
        this.menus = data.data;
      });
    },

    nodeClick(data, node, component) {
        // this.menus = data.data
        // console.log("子组件被点击", node, component, data)
        //
        this.$emit("tree-node-click", data, node, component)
    },
  },
  created() {
    this.getMeus();
  },

  data() {
    return {
      pCid: [],
      menus: [],
      // 默认展开的数组
      title: "", //
      dialogType: "", // add or edit

      defaultProps: {
        children: "children",
        label: "name",
      },
      draggable: false,
      updateNodes: [],
    };
  },
};
</script>

<style>
</style>