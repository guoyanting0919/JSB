﻿<template>
  <div class="categories-wrap">
    <div class="categories-box">
      <div class="categories-container flex-row">
        <div style="border-right: 1px solid #ccc;">
          <div class="buttons-box filter-container">
            <el-button
              :icon="`iconfont icon-${btn.icon}`"
              :type="btn.class"
              size="mini"
              v-for="btn of categoryBtns"
              v-bind:key="btn.Id"
              class="filter-item"
              @click="onBtnClicked(btn.domId)"
            >{{btn.name}}</el-button>
            <div @keyup.13="handleSearchCategoryTypes">
              <el-input
                placeholder="請輸入內容"
                v-model="typesListQuery.key"
                size="mini"
                style="margin-top: 10px;width: 130px;"
              >
                <i slot="prefix" class="el-input__icon el-icon-search"></i>
              </el-input>
              <el-button
                class="filter-item"
                size="mini"
                v-waves
                icon="el-icon-search"
                @click="handleSearchCategoryTypes"
              >搜索</el-button>
            </div>
          </div>
          <el-card shadow="never" class="body-small categories-menu-card">
            <div slot="header" class="clearfix">
              <el-button type="text" style="padding: 0 11px" @click="getAllCategories">全部字典>></el-button>
            </div>
            <el-tree
              :current-node-key="listQuery.TypeId"
              node-key="id"
              :highlight-current="true"
              @node-click="handleNodeClick"
              :data="categoryTypes"
              :expand-on-click-node="false"
              default-expand-all
              :props="categoryTypeProps"
            >
              <span class="custom-tree-node" slot-scope="{ node }">
                <span>
                  <i class="el-icon-menu" style="margin-right: 10px;"></i>
                  {{ node.label }}
                </span>
              </span>
            </el-tree>
          </el-card>
        </div>
        <el-main class="categories-content flex-item">
          <sticky :className="'sub-navbar'">
            <div class="filter-container" style="white-space: nowrap; overflow-x: auto;">
              <el-input
                @keyup.enter.native="handleFilter"
                size="mini"
                style="width: 200px;"
                class="filter-item"
                :placeholder="'名稱'"
                v-model="listQuery.key"
              ></el-input>

              <el-button
                class="filter-item"
                size="mini"
                v-waves
                icon="el-icon-search"
                @click="handleFilter"
              >搜索</el-button>
              <el-button
                :icon="`iconfont icon-${btn.icon}`"
                :type="btn.class"
                size="mini"
                v-for="btn of typesBtns"
                v-bind:key="btn.Id"
                class="filter-item"
                @click="onBtnClicked(btn.domId)"
              >{{btn.name}}</el-button>
            </div>
          </sticky>
          <el-table
            height="calc(100% - 52px - 44px)"
            ref="mainTable"
            :key="tableKey"
            :data="list"
            v-loading="listLoading"
            border
            fit
            highlight-current-row
            style="width: 100%;"
            @row-click="rowClick"
            @selection-change="handleSelectionChange"
          >
            <el-table-column type="selection" align="center" width="55"></el-table-column>
            <template v-for="(headerItem,index) in headerList">
              <el-table-column :label="headerItem.description" min-width="120px" :key="index">
                <template slot-scope="scope">
                  <span>{{scope.row[headerItem.key]}}</span>
                </template>
              </el-table-column>
            </template>
          </el-table>
          <pagination
            v-show="total>0"
            :total="total"
            :page.sync="listQuery.page"
            :limit.sync="listQuery.limit"
            @pagination="handleCurrentChange"
          />
        </el-main>
      </div>
      <el-dialog
        :destroy-on-close="true"
        class="dialog-mini custom-dialog user-dialog"
        width="400px"
        title="添加分組"
        :visible.sync="addTypesDialog"
      >
        <el-form
          ref="categoryTypeForm"
          :model="categoryTypesInfo"
          :rules="categoryRules"
          el="categorys-tayps-form"
          label-width="80px"
        >
          <el-form-item prop="id" label="分類id">
            <el-input size="small" v-model="categoryTypesInfo.id"></el-input>
          </el-form-item>
          <el-form-item prop="name" label="分類名稱">
            <el-input size="small" v-model="categoryTypesInfo.name"></el-input>
          </el-form-item>
        </el-form>
        <div style="text-align:right;margin-top: 10px;">
          <el-button size="small" type="cancel" @click="addTypesDialog = false">取消</el-button>
          <el-button size="small" type="primary" @click="handleAddCategories">確定</el-button>
        </div>
      </el-dialog>
      <el-dialog
        v-el-drag-dialog
        class="dialog-mini"
        width="500px"
        :title="textMap[dialogStatus]"
        :visible.sync="dialogFormVisible"
      >
        <el-form
          :rules="rules"
          ref="dataForm"
          :model="temp"
          label-position="right"
          label-width="100px"
        >
          <el-form-item size="small" :label="'Id'" prop="id">
            <el-input v-model="temp.id" :disabled="true" placeholder="系統自動處理"></el-input>
          </el-form-item>

          <el-form-item size="small" :label="'代碼'" prop="dtCode">
            <el-input v-model="temp.dtCode"></el-input>
          </el-form-item>

          <el-form-item size="small" :label="'名稱'" prop="name">
            <el-input v-model="temp.name"></el-input>
          </el-form-item>

          <el-form-item size="small" :label="'值'" prop="dtValue">
            <el-input v-model="temp.dtValue"></el-input>
          </el-form-item>

          <el-form-item size="small" :label="'是否可用'" prop="enable">
            <el-select class="filter-item" v-model="temp.enable" placeholder="Please select">
              <el-option
                v-for="item in  statusOptions"
                :key="item.key"
                :label="item.display_name"
                :value="item.key"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item size="small" :label="'排序號'">
            <el-input-number v-model="temp.sortNo" :min="0" :max="10"></el-input-number>
          </el-form-item>

          <el-form-item size="small" :label="'描述'" prop="description">
            <el-input v-model="temp.description"></el-input>
          </el-form-item>

          <el-form-item size="small" :label="'所屬分類ID'" prop="typeId">
            <el-select v-model="temp.typeId">
              <el-option
                v-for="(item, index) in categoryTypes"
                :key="index"
                :value="item.id"
                :label="item.name"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer">
          <el-button size="mini" @click="dialogFormVisible = false">取消</el-button>
          <el-button size="mini" v-if="dialogStatus=='create'" type="primary" @click="createData">確認</el-button>
          <el-button size="mini" v-else type="primary" @click="updateData">確認</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import * as categorys from "@/api/categorys";
import waves from "@/directive/waves"; // 水波紋指令
import Sticky from "@/components/Sticky";
import Pagination from "@/components/Pagination";
import elDragDialog from "@/directive/el-dragDialog";
export default {
  name: "category",
  components: { Sticky, Pagination },
  directives: {
    waves,
    elDragDialog,
  },
  data() {
    return {
      multipleSelection: [], // 列表checkbox選中的值
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        // 查詢條件
        page: 1,
        limit: 20,
        key: undefined,
        appId: undefined,
        TypeId: undefined,
      },
      statusOptions: [
        { key: true, display_name: "正常" },
        { key: false, display_name: "停用" },
      ],
      temp: {
        id: "", // 分類表ID（可作為分類的標識）
        dtCode: "",
        name: "", // 名稱
        dtValue: "",
        enable: true, // 是否可用
        sortNo: "", // 排序號
        description: "", // 分類描述
        typeId: "", // 分類類型ID
        extendInfo: "", // 其他信息,防止最後加逗號，可以刪除
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "編輯",
        create: "添加",
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        appId: [
          { required: true, message: "必須選擇一個應用", trigger: "change" },
        ],
        name: [{ required: true, message: "名稱不能為空", trigger: "blur" }],
      },
      downloadLoading: false,
      headerList: [],
      searchCategories: "", // 分類搜索
      addTypesDialog: false,
      categoryTypes: [],
      searchCategoryType: "",
      categoryTypesInfo: {
        id: "",
        name: "",
      },
      categoryRules: {
        name: [
          { required: true, message: "分類名稱不能為空", trigger: "blur" },
        ],
        id: [{ required: true, message: "分類id不能為空", trigger: "blur" }],
      },
      categoryTypeProps: {
        children: "children",
        label: "name",
      },
      typesListQuery: {
        page: 1,
        limit: 99999,
        key: "",
      },
    };
  },
  filters: {
    statusFilter(disable) {
      const statusMap = {
        false: "color-success",
        true: "color-danger",
      };
      return statusMap[disable];
    },
  },
  computed: {
    // 使用對象展開運算符將 getter 混入 computed 對象中
    ...mapGetters(["modulesTree"]),
    categoryBtns() {
      var route = this.$route;
      var elements = route.meta.elements;
      elements =
        elements &&
        elements.length > 0 &&
        elements
          .filter(
            (item) =>
              item.domId === "btnDelCategory" || item.domId === "btnAddCategory"
          )
          .sort((a, b) => {
            return a.sort - b.sort;
          });
      return elements || [];
    },
    typesBtns() {
      var route = this.$route;
      var elements = route.meta.elements;
      elements =
        elements &&
        elements.length > 0 &&
        elements
          .filter(
            (item) =>
              item.domId !== "btnDelCategory" &&
              item.domId !== "btnAddCategory" &&
              item.domId !== "btnRefresh"
          )
          .sort((a, b) => {
            return a.sort - b.sort;
          });
      return elements || [];
    },
    isShowOperation() {
      const route = this.$route;
      const elements = route.meta.elements || [];
      let flag = false;
      elements.forEach((item) => {
        if (item.domId === "btnEdit") {
          flag = true;
        }
      });
      return flag;
    },
  },
  created() {
    this.getList();
    this.loadCategoryTypes();
  },
  methods: {
    getAllCategories() {
      this.listQuery.TypeId = "";
      this.listQuery.page = 1;
      this.getList();
    },
    rowClick(row) {
      this.$refs.mainTable.clearSelection();
      this.$refs.mainTable.toggleRowSelection(row);
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    onBtnClicked: function (domId) {
      console.log("you click:" + domId);
      switch (domId) {
        case "btnAdd":
          this.handleCreate();
          break;
        case "btnEdit":
          if (this.multipleSelection.length !== 1) {
            this.$message({
              message: "只能選中一個進行編輯",
              type: "error",
            });
            return;
          }
          this.handleUpdate(this.multipleSelection[0]);
          break;
        case "btnDel":
          if (this.multipleSelection.length < 1) {
            this.$message({
              message: "至少刪除一個",
              type: "error",
            });
            return;
          }
          this.handleDelete(this.multipleSelection);
          break;
        case "btnAddCategory":
          this.addTypesDialog = true;
          break;
        case "btnDelCategory":
          this.handleDeleteCategories();
          break;
        default:
          break;
      }
    },
    getList() {
      this.listLoading = true;
      categorys.getList(this.listQuery).then((response) => {
        response.columnHeaders.forEach((item) => {
          item.key =
            item.key.substring(0, 1).toLowerCase() + item.key.substring(1);
        });
        this.headerList = response.columnHeaders.filter((u) => u.browsable);
        this.list = response.data;
        this.total = response.count;
        this.listLoading = false;
      });
    },
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    handleSizeChange(val) {
      this.listQuery.limit = val;
      this.getList();
    },
    handleCurrentChange(val) {
      this.listQuery.page = val.page;
      this.listQuery.limit = val.limit;
      this.getList();
    },
    resetTemp() {
      this.temp = {
        id: "",
        name: "",
        enable: true,
        dtCode: "",
        dtValue: "",
        sortNo: "",
        description: "",
        typeId: "",
        extendInfo: "",
      };
    },
    handleCreate() {
      // 彈出添加框
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    createData() {
      // 保存提交
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          categorys.add(this.temp).then(() => {
            this.list.unshift(this.temp);
            this.dialogFormVisible = false;
            this.$notify({
              title: "成功",
              message: "創建成功",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleUpdate(row) {
      // 彈出編輯框
      this.temp = Object.assign({}, row); // copy obj
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      // 更新提交
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          categorys.update(tempData).then(() => {
            for (const v of this.list) {
              if (v.id === this.temp.id) {
                const index = this.list.indexOf(v);
                this.list.splice(index, 1, this.temp);
                break;
              }
            }
            this.dialogFormVisible = false;
            this.$notify({
              title: "成功",
              message: "更新成功",
              type: "success",
              duration: 2000,
            });
          });
        }
      });
    },
    handleDelete(rows) {
      // 多行刪除
      categorys.del(rows.map((u) => u.id)).then(() => {
        this.$notify({
          title: "成功",
          message: "刪除成功",
          type: "success",
          duration: 2000,
        });
        rows.forEach((row) => {
          const index = this.list.indexOf(row);
          this.list.splice(index, 1);
        });
      });
    },
    // 添加分類
    handleAddCategories() {
      this.$refs["categoryTypeForm"].validate((valid) => {
        if (valid) {
          categorys.addType(this.categoryTypesInfo).then(() => {
            this.$notify({
              title: "成功",
              message: "添加成功",
              type: "success",
              duration: 2000,
            });
            this.addTypesDialog = false;
            this.categoryTypes.push(this.categoryTypesInfo);
          });
        }
      });
    },
    // 刪除分類
    handleDeleteCategories() {
      if (!this.listQuery.TypeId) {
        this.$message({
          message: "請選擇需要刪除的分類",
          type: "error",
        });
        return;
      }
      const ids = [this.listQuery.TypeId];
      categorys.delType(ids).then(() => {
        this.$notify({
          title: "成功",
          message: "刪除成功",
          type: "success",
          duration: 2000,
        });
        this.categoryTypes = this.categoryTypes.filter(
          (item) => item.id !== this.listQuery.TypeId
        );
      });
    },
    // 搜索分類
    handleSearchCategoryTypes() {
      this.typesListQuery.page = 1;
      this.loadCategoryTypes();
    },
    // 獲取分類列表
    loadCategoryTypes() {
      categorys.loadType(this.typesListQuery).then((res) => {
        this.categoryTypes = [...res.data];
      });
    },
    handleNodeClick(val) {
      this.listQuery.TypeId = val.id;
      this.getList();
    },
  },
};
</script>
<style rel="stylesheet/scss" lang="scss">
.dialog-mini .el-select {
  width: 100%;
}
.categories-wrap {
  width: 100%;
  height: calc(100vh - 84px);
  box-sizing: border-box;
  padding: 10px;
  .categories-menu-card {
    height: calc(100% - 86px);
    overflow: auto;
    .el-card__body {
      height: auto;
    }
  }
}
.categories-box {
  height: 100%;
  background: #fff;
  box-sizing: border-box;
  .categories-container {
    width: 100%;
    height: 100%;
    border: 1px solid #ccc;
    box-sizing: border-box;
    position: relative;
    .categories-item {
      height: 30px;
      line-height: 30px;
      padding: 0 10px !important;
    }
  }
  .buttons-box {
    padding: 10px 10px;
    background: #f5f5f5;
  }
  .categories-content {
    height: 100%;
    padding: 0;
    .buttons-box {
      text-align: right;
    }
  }
}
</style>
