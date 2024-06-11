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
            @click="() => append(node, data)"
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
      }
    }
  },
  methods: {
    getCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/listTree"),
        method: "get"
      }).then(({ data }) => {
        // console.log("成功获取的类别数据：", data.data)
        this.data = data.data
      })
    },
    append(node, data) {
      this.add(node, data)
      console.log("添加", node, data)
    },
    remove(node, data) {
      this.open(node, data)
      console.log("删除", data, node)
    },
    open(node, data) {
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
    },
    add(node, data) {
      let catLevel = 0
      if (data.catLevel == 1) {
        catLevel = 2
      } else if (data.catLevel == 2) {
        catLevel = 3
      }
      this.$prompt("请输入分类名称", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(({ value }) => {
          let categoryJson = {
            name: value,
            parentCid: data.catId,
            catLevel: catLevel,
            productCount: 0,
            productUnit: null,
            showStatus: 1,
            sort: 0
          }
          this.$http({
            url: this.$http.adornUrl("/product/category/save"),
            method: "post",
            data: this.$http.adornData(categoryJson, false)
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "添加成功",
                type: "success"
              })

              //刷新页面
              this.getCategory()
              //保持展示的页面还是当前删除的父节点
              this.expandKeys = [node.data.catId]
            } else {
              this.$message.error(data.msg)
            }
          })
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "取消输入"
          })
        })
    }
  },
  created() {
    this.getCategory()
  }
}
</script>

<style></style>
