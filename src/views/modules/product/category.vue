<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>

    <el-button v-if="draggable" @click="batchSave()"> 批量保存</el-button>
    <el-button @click="batchDel()" type="danger">批量删除</el-button>

    <el-tree
      ref="menuTree"
      :data="menus"
      :default-expanded-keys="expandKey"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      @node-drop="handleDrop"
      :draggable="draggable"
      :allow-drop="allowDrop"
      node-drop
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

          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
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
      :before-close="handleClose"
      :close-on-click-modal="false"
    >
      <el-form :model="category" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pCid: [],
      menus: [],
      // 默认展开的数组
      title: "", //
      expandKey: [],
      dialogVisible: false,
      dialogType: "", // add or edit
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      defaultProps: {
        children: "children",
        label: "name",
      },
      draggable: false,
      updateNodes: [],
    };
  },

  created() {
    this.getMeus();
  },

  methods: {
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },

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

    handleNodeClick(data) {
      // console.log(data);
    },

    append(data) {
      this.dialogVisible = true;
      this.dialogType = "add";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      // console.log(data);
      this.title = "添加分类";

      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      this.category.name = "";
    },

    batchSave() {
      // 3, 更新节点
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "保存成功",
          type: "success",
        });
        // console.log(this.pCid)
        this.getMeus();
        this.expandKey = [this.pCid];
        this.updateNodes = [];
        this.pCid = [];
      });
    },

    batchDel() {
      let checkNodes = this.$refs.menuTree.getCheckedNodes();
      // console.log(checkNodes)
      let catIds = []
      for (let i = 0; i < checkNodes.length; i ++ ) catIds.push(checkNodes[i].catId)
      this.$confirm(`是否批量删除【${catIds}】当前菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({ data }) => {
          this.$message({
            message: "删除成功",
            type: "success",
          });
          this.getMeus()
        });
      }).catch(() => {

      })
    },

    // 修改 方法
    editCategory(data) {
      var { name, catId, icon, productUnit, parentCid } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ name, catId, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          message: "修改成功",
          type: "success",
        });
        this.dialogVisible = false;
        this.getMeus();
        // 设置默认展开的菜单
        this.expandKey = [parentCid];
      });
    },

    edit(data) {
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.title = "修改分类";

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("需要回显的数据", data.data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },

    submitData() {
      // 根据 当前 dialogType 来判断 add or edit
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    allowDrop(draggingNode, dropNode, type) {
      // console.log(draggingNode, dropNode, type)
      // console.log(draggingNode.data, " ====== ")
      // console.log()
      return this.countNodeLevel(draggingNode.data) + dropNode.level <= 3;
    },

    countNodeLevel(node) {
      let nowLevel = 0;
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          let t = this.countNodeLevel(node.children[i]) + 1;
          nowLevel = nowLevel > t ? nowLevel : t;
        }
      }
      return nowLevel;
    },

    handleDrop(draggingNode, dropNode, dropType, ev) {
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);

      // 2, 当前拖拽节点的 最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          // 当前节点的 层级发生变化
          if (siblings[i].level != draggingNode.level) {
            catLevel = siblings[i].level;
            this.updateChildNodes(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }

      // console.log(this.updateNodes, " ===== ")
      // console.log('tree drop: ', draggingNode, dropNode, dropType);
    },

    updateChildNodes(node) {
      if (node.childNodes.length == 0) return;
      for (let i = 0; i < node.childNodes.length; i++) {
        let cNode = node.childNodes[i].data;
        this.updateNodes.push({
          catId: cNode.catId,
          catLevel: node.childNodes[i].level,
        });
        this.updateChildNodes(node.childNodes[i]);
      }
    },

    // 添加三级 分类的 方法
    addCategory() {
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "保存成功",
          type: "success",
        });
        this.dialogVisible = false;
        this.getMeus();
        // 设置默认展开的菜单
        this.expandKey = [this.category.parentCid];
      });
      // console.log(this.category)
    },

    remove(node, data) {
      var ids = [data.catId];

      this.$confirm(`是否删除【${data.name}】当前菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "删除成功",
              type: "success",
            });
            this.getMeus();
            this.expandKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>

<style></style>