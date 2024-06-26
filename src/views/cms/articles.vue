<template>
  <div class="mod-demo__news">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item style="width: 190px;">
        <el-tree-select clearable :placeholder="$t('service.selectCategory')" v-model="state.dataForm.categoryId" ref="articlesTree" :data="categories" :props="{ label: 'name', children: 'children' }" :render-after-expand="false" accordion :expand-on-click-node="true" node-key="id" :highlight-current="true"/>
      </el-form-item>
      <el-form-item style="width: 190px;">
        <el-select v-model="state.dataForm.status">
          <el-option :value="'0'" :label="'All'"></el-option>
          <el-option :label="data.dictLabel" v-for="data in getDictDataList(useAppStore().state.dicts, 'articles_status')" :key="data.dictValue" :value="data.dictValue">{{ data.dictLabel }}</el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.title" :placeholder="$t('news.title')" clearable></el-input>
      </el-form-item>
      <el-form-item style="width: 300px;">
        <el-date-picker v-model="state.daterange" type="daterange" value-format="YYYY-MM-DD" :range-separator="$t('datePicker.range')" :start-placeholder="$t('datePicker.start')" :end-placeholder="$t('datePicker.end')"></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" @sort-change="state.dataListSortChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="title" :label="$t('news.title')" header-align="center" align="center" :show-overflow-tooltip="true"></el-table-column>
      <el-table-column prop="status" :label="'Status'" header-align="center" align="center" width="150">
        <template v-slot="scope">
          {{ state.getDictLabel("articles_status", scope.row.status) }}
        </template>
      </el-table-column>
      <el-table-column prop="categoryName" :label="'Category'" header-align="center" align="center"></el-table-column>
      <el-table-column prop="pubDate" :label="'PublishTime'" header-align="center" align="center" width="180"></el-table-column>
      <el-table-column prop="author" :label="'Creator'" header-align="center" align="center" width="180"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import {nextTick, reactive, ref, toRefs, watch} from "vue";
import AddOrUpdate from "./articles-add-or-update.vue";
import baseService from "@/service/baseService";
import {getDictDataList} from "@/utils/utils";
import {useAppStore} from "@/store";

const categories = ref([]);
const view = reactive({
  getDataListURL: "/cms/articles/page",
  getDataListIsPage: true,
  deleteURL: "/cms/articles",
  deleteIsBatch: true,
  daterange: null,
  dataForm: {
    title: "",
    categoryId: "",
    status: "0",
    startDate: null as number | null,
    endDate: null as number | null
  }
});

const state = reactive({...useView(view), ...toRefs(view)});

watch(
  () => state.daterange,
  (val: number[] | null) => {
    if (val === null) {
      state.dataForm.startDate = null;
      state.dataForm.endDate = null;
    } else {
      state.dataForm.startDate = val[0];
      state.dataForm.endDate = val[1];
    }
  }
);

const addOrUpdateRef = ref();
const addKey = ref(0);
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};

const init = () => {
  baseService.get("/cms/categories/public/list?pid=100").then((res) => {
    categories.value = res.data;
  });
}
init()
</script>
