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

           <el-button
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            Edit
          </el-button>
          
        </span>
      </span>
    </el-tree>

    <el-dialog title="收货地址" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="活动名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="活动区域" :label-width="formLabelWidth">
          <el-select v-model="form.region" placeholder="请选择活动区域">
            <el-option label="区域一" value="shanghai"></el-option>
            <el-option label="区域二" value="beijing"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false"
          >确 定</el-button
        >
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
      menus: [],
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



  },

  created() {
    this.getMenus();
  },
};
</script>

<style scoped>
</style>
