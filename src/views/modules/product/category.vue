<template>
  <!--  :expand-on-click-node    show-checkbox    node-key="catId" 具体功能去官网查看
第一个button按钮，v-if="node.level<=2" 这样只有1,2级菜单才能使用添加功能
第二个button按钮，v-if="node.childNodes.length==0"  只有没有子节点的菜单才能使用删除功能
在methods中添加了append和remove方法用来测试
-->
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button type="success" @click="batchSave" v-if="draggable"
      >批量保存</el-button
    >
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
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

          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  comments: {},
  props: {},

  data() {
    return {
      // 接受拖拽之后更新的父节点信息
      pCid: [],
      // 开启拖拽功能
      draggable: false,
      // 保存所有要修改的节点信息，拖拽功能后节点的信息，用于拖拽节点的数据持久化
      updateNodes: [],
      // 用来接收拖拽功能判断最大catLevel
      maxLevel: 0,
      menus: [],
      // 控制对话框弹出可见
      dialogFormVisible: false,
      // 控制对话框提示消息
      title: "",
      // 控制对话框点击确定按钮调用哪个方法
      dialogType: "",
      // 保存菜单提交的数据
      category: {
        name: "",
        catId: null,
        parentCid: 0,
        icon: "",
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
      },
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },

  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // 注意then里面的data是{data}解构赋值！！！！
        console.log("successly get data...", data.data);
        return (this.menus = data.data);
      });
    },

    append(data) {
      console.log("append", data);
      this.dialogType = "add";
      this.dialogFormVisible = true;
      this.title = "添加分类";
      // this.category={};
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.showStatus = 1;
      this.category.sort = 0;
      this.category.parentCid = data.catId;

      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.parentCid = data.catId;
    },

    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单分类保存成功",
          type: "success",
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      console.log("remove", node, data);

      var ids = [data.catId];

      // 消息弹框，但删除成功时使用弹窗提醒

      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          // 点击确定之后，成功的操作
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            // 弹出信息框，提示删除成功在页面显示
            this.$message({
              showClose: true,
              message: "删除成功",
              type: "success",
            });
            // 在控制台显示消息
            console.log("删除成功");

            // 刷新三级菜单目录
            this.getMenus();

            // 设置默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          // 点击取消之后的操作，暂无
        });
    },

    edit(data) {
      this.dialogFormVisible = true;
      this.dialogType = "edit";
      this.title = "修改分类";

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("要回显的数据", data);
        this.category.catId = data.data.catId;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },

    editCategory() {
      console.log("editCategory method", this.category);
      var { catId, name, productUnit, icon } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, productUnit, icon }, false),
      }).then(({ data }) => {
        this.$message({
          message: "修改成功",
          type: "success",
        });

        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    allowDrop(draggingNode, dropNode, type) {
      // 避免多次使用拖拽之后，maxLevel发生值不准确,如果没有子节点，那么maxLevel值就等于本身的catId
      this.maxLevel = draggingNode.level;

      // 三个节点的意义：draggingNode（当前拖动的节点）、dropNode（目标节点）、type（放置的位置，在其上、下或是成为其子节点）

      console.log("allowDrop", draggingNode, dropNode, type);

      // 调用函数
      this.countNodeLevel(draggingNode);

      // 最大深度就是指，被拖动节点的有几层，比如拖动的几点是一级节点（存在二级和三级节点），那么它的层数（也就是深度）就是3
      // 计算就是：3-1+1 ，（maxLevel）三级catLevel - （draggingNode.catLevel）一级catLevel + 1 ，其他节点都可以计算得出。
      let deep = this.maxLevel - draggingNode.level + 1;
      console.log("深度：", deep);

      // 判断能不能拖动。只要拖动节点的深度+目标节点的Level不大于3即可（因为是三级分类，所以最大catLevel不能大于3）
      // 如果是往某个节点里面拖，return (deep+dropNode.level)<=3;如果往某个节点上下拖，就相当于往目标节点的父节点里面拖
      if (type == "inner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }

      return;
    },

    //查找节点及其所有子节点的最大 catLevel，并将其复制给maxLevel
    countNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          // 递归调用，查找所有子节点的catLevel
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },

    // 拖拽节点之后，将节点属性进行重新赋值
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);

      let pCid = 0;
      let sibings = null;
      // this.updateNodes = [];

      // 1、当前节点的最新的父节点id
      // 如果拖拽的方式是在目标节点前后，那么最新的父节点id就是dropNode的父节点
      // 如果拖拽的范式是进入目标节点，那么最新的父节点就是目标节点dropNode

      //  自己写的逻辑
      // if(dropNode=="inner"){
      //   pCid=dropNode.data.catId;
      // }else{
      //   pCid=dropNode.data.parentCid;
      // };

      // 三元运算符写法 let pCid=dropType=="inner"?dropNode.data.catId:dropNode.parent.data.catId;

      // 项目写法:当前节点的最新的父节点id以及最新的兄弟节点
      if (dropType == "before" || dropType == "after") {
        // 如果将某个节点拖拽至一级菜单，此时父id不是某一个节点，而是一个Array，这时候我们需要把draggingNode的父节点设置为0
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;

        // 更新后的兄弟节点
        sibings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        sibings = dropNode.childNodes;
      }

      this.pCid.push(pCid);

      // 2、当前拖拽节点的最新排序更新
      // 3、当前拖拽节点的最新层级更新
      for (let i = 0; i < sibings.length; i++) {
        // 判断遍历的节点是不是拖拽的节点，如果是，就要改变它的父节点id，为最新的pCid
        if (sibings[i].data.catId == draggingNode.data.catId) {
          // 更新拖动节点的层级
          let catLevel = draggingNode.level;
          // 如果拖动之后层级发生变化，就更新拖动节点以及子节点的层级
          if (sibings[i].level != catLevel) {
            // 拖拽的节点，level更新
            catLevel = sibings[i].level;

            // 子节点level更新，调用updateChildLevel方法
            this.updateChildLevel(sibings[i]);
          }

          // 将兄弟节点进行排序，数据库中的catId为兄弟结案的catId，sort值为i，并更新父节点
          this.updateNodes.push({
            catId: sibings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        }

        // 如果遍历的节点不是拖拽的节点，只排序即可
        else {
          this.updateNodes.push({ catId: sibings[i].data.catId, sort: i });
        }
      }

      // 打印要更新的节点数据信息
      console.log("updateNodes", this.updateNodes);

      // for (let i = 0; i < this.updateNodes.length; i++) {
      //   var { catId, sort } = this.updateNodes[i];
      //   console.log("updateNodes", { catId, sort });
      // }
    },

    // 递归更新节点的所有level
    updateChildLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          // 递归调用
          this.updateChildLevel(node.childNodes[i]);
        }
      }
    },

    // 批量保存
    batchSave() {
      // 发送跟后端进行数据持久化保存
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单节点顺序数据更新成功",
          type: "success",
        });
        this.getMenus();
        this.expandedKey = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        this.pCid = [];
      });
    },
    // TODO: 批量删除
    batchDelete() {
      // 要删除的catId数组
      let catIds = [];
      let checkedNodesNames = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("checkedNodes", checkedNodes);
      // 遍历得到所有的被选中的节点id
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
        checkedNodesNames.push(checkedNodes[i].name);
      }
      this.$confirm(`是否批量删除【${checkedNodesNames}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              message:"菜单批量删除成功",
              type:"success"
            });
            this.getMenus();
          });
        })
        .catch(() => {});
    },
  },

  created() {
    this.getMenus();
  },
};
</script>

<style scoped>
</style>
