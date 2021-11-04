<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
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
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
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
//          <!-- v-if="node.level <= 2" -->
//          <!-- v-if="node.childrenNodes.length == 0" -->
// 这里的树形结构，data="menus"是调用data()类中的数组元素
//handleNodeClick为点击一次结点调用的函数
//label返回的是树形结构结点的值，用k:v
//data.page为CategoryController类中/product/category/list/tree中list方法返回的page信息
//expand-on-click-node为点击是否展开子菜单，false为不展开
//这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    return {
      draggable: false,
      pCid: [],
      draggable: false,
      updateNodes: [],
      maxlevel: 0,
      title: "",
      dialogType: {}, //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
        catId: null,
      },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
  },

  //计算属性 类似于 data 概念
  computed: {},
  //监控 data 中的数据变化
  watch: {},
  //方法集合
  methods: {
    getMenus() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据。。。", data.page);
        // console.log(data.id);
        this.menus = data.page;
      });
    },
    batchSave() {
      //批量拖拽
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单顺序修改成功",
          type: "success",
        });
        //刷新菜单并展开设置默认展开菜单
        //刷新出新的菜单
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = this.pCid;
        // console.log(this.expandedKey);

        //将updateNodes、maxlevel初始化，否则影响菜单拖拽
        this.updateNodes = [];
        this.maxlevel = 0;
        // this.pCid=[];
      });
    },

    batchDelete() {
      //使用refs表示树的唯一标识中方法getCheckNodes，得到当前勾选的结点组成的数组
      let catIds = [];
      let catNames=[];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      console.log("被选中的结点", checkedNodes);
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId);
        catNames.push(checkedNodes[i].name);
      }
      this.$confirm(`是否批量删除当前【${catNames}】菜单?`, `提示`, {
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
              message: "菜单批量删除成功",
              type: "success",
            });
            //刷新出新的菜单
            this.getMenus();
            //设置默认展开的菜单
            // this.expandedKey = [node.parent.data.catId];
            // console.log(this.expandedKey);
          });
        })
        .catch(() => {});
    },

    append(data) {
      console.log("append", data);
      this.title = "添加分类";
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * +1;
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },

    //添加三级分类
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = [this.category.parentCid];
        // console.log(this.expandedKey);
      });
      console.log(this.category);
    },

    remove(node, data) {
      var ids = [data.catId];
      var name = data.name;
      this.$confirm(`是否删除当前【${name}】菜单?`, `提示`, {
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
              message: "菜单删除成功",
              type: "success",
            });
            //刷新出新的菜单
            this.getMenus();
            //设置默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
            // console.log(this.expandedKey);
          });
        })
        .catch(() => {});

      console.log("remove", node, data);
    },

    edit(data) {
      this.dialogType = "edit";
      this.title = "修改分类";
      console.log("要修改的数据", data);
      this.dialogVisible = true;
      //发送请求获取当前节点最新数据，考虑到并发情况下数据脏读问题
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        // params: this.$http.adornParams({}),
      }).then(({ data }) => {
        //请求成功
        console.log("要回显的数据", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;

        //回显其他字段
        this.category.catLevel = data.data.catLevel;
        this.category.showStatus = data.data.showStatus;
        this.category.sort = data.data.sort;
        console.log("要回显的数据", this.category);
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
    //修改三级分类数据    { catId: catId, naem: name, icon: icon, productUnit: productUnit }
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      // var data=
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(
          { catId: catId, name: name, icon: icon, productUnit: productUnit },
          false
        ),
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });
        //关闭对话框
        this.dialogVisible = false;
        this.getMenus();
        //设置默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    //拖拽后将数据存入数据库
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("tree drop: ", dropNode.label, dropType);
      //当前结点最新的父结点id
      let pCid = 0;
      let siblings = null; //兄弟结点
      //拖拽后结点上下层级发生改变
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
      //当前拖拽结点的最新顺序i
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          //如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            //当前节点的层级发生变化
            catLevel = siblings[i].level;
            //修改他子节点的层级
            this.updateChildNodeLevel(siblings[i]);
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
      //当前拖拽结点的最新层级level
      console.log("updateNodes", this.updateNodes);
      // this.batchSave();
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },

    allowDrop(draggingNode, dropNode, type) {
      //1、被拖动的当前节点以及所在的父节点总层数不能大于3
      //1> 被拖动的当前节点层
      //这里的深度值得是  拖拽部分有多少层级

      // 另外在计算maxLevel时不能直接就调用countNodeLevel计算;
      // 当拖动末端节点时不满足其计算条件, 就会使用上次的maxLevel值造成异常;
      // 此时maxLevel要直接设置为拖动节点的level;

      console.log("allowDrop", draggingNode.data, dropNode, type);
      this.countNodeLevel(draggingNode.data);
      //当前正在拖动的节点所在的深度+父节点所在的深度不大于3
      // this.maxlevel

      var deep = Math.abs(this.maxlevel - draggingNode.data.catLevel) + 1;
      // console.log("深度:", deep);

      //出现BUG：无法每次成功构建三级菜单——》没有及时读取数据库信息
      if (type == "inner") {
        console.log(
          `this.maxLevel:${this.maxlevel};draggingNode.data.catLevel:${draggingNode.data.catLevel};dropNode.level:${dropNode.level}`
        );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
      // return deep + dropNode;
    },
    countNodeLevel(node) {
      //找到所有子节点，求出最大深度
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxlevel) {
            this.maxlevel = node.children[i].catLevel;
          }
          this.countNodeLevel(node.children[i]);
        }
      }
    },
  },
  //生命周期 - 创建完成（可以访问当前 this 实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted() {},
  //生命周期 - 创建之前
  beforeCreate() {},
  //生命周期 - 挂载之前
  beforeMount() {},
  //生命周期 - 更新之前
  beforeUpdate() {},
  //生命周期 - 更新之后
  updated() {},
  //生命周期 - 销毁之前
  beforeDestroy() {},
  //生命周期 - 销毁完成
  destroyed() {},
  //如果页面有 keep-alive 缓存功能，这个函数会触发
  activated() {},
};
</script>
<style  scoped>
/*  @import url(); 引入公共 css 类 */
</style>