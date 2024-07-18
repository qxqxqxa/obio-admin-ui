<template>
  <div class="mod-cms__cmsinquiry">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="state.dataForm.email" placeholder="Email" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.phone" placeholder="Phone" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-input v-model="state.dataForm.company" placeholder="Company" clearable></el-input>
      </el-form-item>
       <el-form-item>
        <el-input v-model="state.dataForm.title" placeholder="Title" clearable></el-input>
      </el-form-item>

      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <!-- <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item> -->
      <!-- <el-form-item>
        <el-button  type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item> -->
      <el-form-item>
        <el-button type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="firstName" label="FirstName" header-align="center" align="center"></el-table-column>
      <el-table-column prop="lastName" label="LastName" header-align="center" align="center"></el-table-column>
      <el-table-column prop="company" label="Company" header-align="center" align="center"></el-table-column>
      <el-table-column prop="title" label="Title" header-align="center" align="center"></el-table-column>
      <el-table-column prop="email" label="Email" header-align="center" align="center"></el-table-column>
      <el-table-column prop="phone" label="Phone" header-align="center" align="center"></el-table-column>
      <el-table-column prop="inquiredItem" label="InquiredItem" header-align="center" align="center"></el-table-column>
      <el-table-column prop="comment" label="Comment" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="fromEmail" label="" header-align="center" align="center"></el-table-column> -->
      <!-- <el-table-column prop="creator" label="创建者" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="createDate" label="CreateDate" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="updater" label="更新者" header-align="center" align="center"></el-table-column> -->
      <!-- <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column> -->
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button  type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("Detail") }}</el-button>
          <el-button  type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./cmsinquiry-add-or-update.vue";

const view = reactive({
  getDataListURL: "/cms/inquiry/page",
  getDataListIsPage: true,
  exportURL: "/cms/inquiry/export",
  deleteURL: "/cms/inquiry",
  deleteIsBatch: true,
  dataForm: {
    company:"",
    email:"",
    phone:"",
    title:"",
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });


const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};
</script>