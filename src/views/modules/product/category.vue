<template>
  <!--  :expand-on-click-node    show-checkbox    node-key="catId" 具体功能去官网查看
第一个button按钮，v-if="node.level<=2" 这样只有1,2级菜单才能使用添加功能
第二个button按钮，v-if="node.childNodes.length==0"  只有没有子节点的菜单才能使用删除功能
在methods中添加了append和remove方法用来测试
-->
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

          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogFormVisible" :close-on-click-modal=false>
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
import categoryVue from "../../../../文档/category.vue";
export default {
  comments: {},
  props: {},

  data() {
    return {
      menus: [],
      dialogFormVisible: false,
      title: "",
      dialogType: "",
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
        this.category.parentCid=data.data.parentCid;
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
  },

  created() {
    this.getMenus();
  },
};
</script>

<style scoped>
</style>
