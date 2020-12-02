<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="组名" prop="attrGroupName">
        <el-input
          v-model="dataForm.attrGroupName"
          placeholder="组名"
        ></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="组图标"></el-input>
      </el-form-item>
      <el-form-item label="所属分类id" prop="catelogId">
        <!-- <el-input v-model="dataForm.catelogId" placeholder="所属分类id"></el-input> -->
        <!-- todo:P73 分组新增和级联选择器 -->
        <el-cascader
          v-model="dataForm.catelogIds"
          :options="categorys"
          :props="props"
        ></el-cascader>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      // 接收返回三级菜单的数据
      categorys:[],
      // 用于展示的选项
      props:{
        label:"name",
        value:"catId",
        children:"children"
      },
      visible: false,
      dataForm: {
        attrGroupId: 0,
        attrGroupName: "",
        sort: "",
        descript: "",
        icon: "",
         // todo: P73 存放级联选择器选中的所有catId（一级分类，二级分类，三级分类），
        //  但是所属分类的id应该是范围最小的一个，即最后一个，提交最后一个即可
        catelogIds: [],
        catelogId:0
      },
      dataRule: {
        attrGroupName: [
          { required: true, message: "组名不能为空", trigger: "blur" },
        ],
        sort: [{ required: true, message: "排序不能为空", trigger: "blur" }],
        descript: [
          { required: true, message: "描述不能为空", trigger: "blur" },
        ],
        icon: [{ required: true, message: "组图标不能为空", trigger: "blur" }],
        catelogId: [
          { required: true, message: "所属分类id不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    // todo: P73
    getCategorys(){
        this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        // 注意then里面的data是{data}解构赋值！！！！
        console.log("successly get data...", data.data);
        return (this.categorys = data.data);
      });
    },

    init(id) {
      this.dataForm.attrGroupId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attrgroup/info/${this.dataForm.attrGroupId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.attrGroupName = data.attrGroup.attrGroupName;
              this.dataForm.sort = data.attrGroup.sort;
              this.dataForm.descript = data.attrGroup.descript;
              this.dataForm.icon = data.attrGroup.icon;
              this.dataForm.catelogId = data.attrGroup.catelogId;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/product/attrgroup/${
                !this.dataForm.attrGroupId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              attrGroupId: this.dataForm.attrGroupId || undefined,
              attrGroupName: this.dataForm.attrGroupName,
              sort: this.dataForm.sort,
              descript: this.dataForm.descript,
              icon: this.dataForm.icon,
               // todo: P73 提交catIds数组里面的最后一个值
              catelogId: this.dataForm.catelogIds[this.dataForm.catelogIds.length-1],
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
  created(){
    this.getCategorys();
  }
};
</script>
