<template>
  <div>
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandKeys"
      show-checkbox
      node-key="catId"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加
          </el-button>
          <el-button
            v-if="data.children.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <!-- 打开嵌套表单的 Dialog -->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%"
    >
      <el-form :model="categoryForm">
        <el-form-item label="类别名称">
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="categoryForm.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="categoryForm.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialog">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      data: [],
      expandKeys: [],
      defaultProps: {
        children: "children",
        label: "name"
      },
      categoryForm: {
        name: "",
        icon: "",
        parentCid: 0,
        catLevel: 0,
        productCount: 0,
        productUnit: null,
        showStatus: 1,
        sort: 0
      },
      dialogVisible: false,
      dialogType: "add"
    }
  },
  methods: {
    //获取所有类别数据
    getCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/listTree"),
        method: "get"
      }).then(({ data }) => {
        // console.log("成功获取的类别数据：", data.data)
        this.data = data.data
      })
    },

    //添加类别，显示弹窗
    append(data) {
      this.dialogVisible = true
      console.log("添加", data)
      // console.log("添加", data);
      this.categoryForm.parentCid = data.catId // 添加的类别对应的父菜单
      this.categoryForm.catLevel = data.catLevel + 1 // 设置添加类别的层级
      this.categoryForm.showStatus = 1 // 菜单的显示状态  1 显示  0 被删除
      this.categoryForm.sort = 0 // 排序 默认给 0
      this.categoryForm.productCount = 0
    },

    //删除类别
    remove(node, data) {
      this.$confirm(`是否确认删除【${data.name}】记录?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          let ids = [data.catId]
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "删除成功",
                type: "success"
              })

              //刷新页面
              this.getCategory()
              //保持展示的页面还是当前删除的父节点
              this.expandKeys = [node.parent.data.catId]
            } else {
              this.$message.error(data.msg)
            }
          })
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          })
        })
      console.log("删除", data, node)
    },

    //弹框点击确定发起添加请求到数据库
    addDialog() {
      // 添加三级分类的类别信息
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.categoryForm, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "数据添加成功",
            type: "success"
          })
          this.dialogVisible = false // 关闭弹出窗口
          this.categoryForm.name = ""
          this.categoryForm.icon = ""
          this.categoryForm.productUnit = ""
          // 重新加载所有的菜单数据
          this.getCategory()
          // 设置默认展开的父节点信息
          this.expandKeys = [this.categoryForm.parentCid]
        } else {
          this.$message.error(data.msg)
        }
      })
    }
  },
  created() {
    this.getCategory()
  }
}
</script>

<style></style>
