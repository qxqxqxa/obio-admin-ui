<template>
  <div class="mod-oss__oss">
    <el-row>
      <el-col :span="4">
        <el-tree style="max-width: 600px" :data="treeData" :props="defaultProps" @node-click="handleNodeClick"/>
      </el-col>
      <el-col :span="20">
        <el-form :inline="true" :model="state.dataForm">
          <el-form-item>
            <el-input v-model="state.dataForm.name" placeholder="Name"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="state.getDataList()">{{ $t("query") }}</el-button>
          </el-form-item>
          <el-form-item v-if="false">
            <el-button type="primary" @click="configHandle()">{{ $t("oss.config") }}</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="uploadHandle()">{{ $t("add") }}</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
            <input id="selectInput" style="height: 0;border: 0;border-top-style: solid;border-right-style: solid;"/>
          </el-form-item>
        </el-form>
        <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" @sort-change="state.dataListSortChangeHandle" style="width: 100%">
          <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
          <el-table-column prop="name" :label="$t('oss.name')" header-align="center" align="center"></el-table-column>
          <el-table-column prop="url" :label="$t('oss.url')" header-align="center" align="center">
            <template v-slot="scope">
              <el-popover placement="top-start" trigger="hover">
                <img :src="scope.row.url" style="width: 90%;">
                <template #reference>
                  {{ scope.row.url }}
                </template>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column prop="createDate" :label="$t('oss.createDate')" sortable="custom" header-align="center" align="center" width="180"></el-table-column>
          <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
            <template v-slot="scope">
              <el-button type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
              <el-button type="primary" link @click="copyText(scope.row.url)">Copy</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"></el-pagination>
      </el-col>
      <!-- 弹窗, 云存储配置 -->
      <config v-if="state.configVisible" ref="configRef"></config>
      <!-- 弹窗, 上传文件 -->
      <upload v-if="state.uploadVisible" ref="uploadRef" @refreshDataList="state.getDataList"></upload>
    </el-row>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import {nextTick, reactive, ref, toRefs} from "vue";
import Config from "./oss-config.vue";
import Upload from "./oss-upload.vue";
import baseService from "@/service/baseService";
import {ElMessage} from "element-plus";


const treeData = ref([])

const defaultProps = {
  children: 'children',
  label: 'label',
}
const handleNodeClick = (data: any, b: any, c: any) => {
  console.log()
  state.dataForm.url = (b.parent.label || '') + '/' + data.label
  state.getDataList()
}
const configRef = ref();
const uploadRef = ref();

const view = reactive({
  getDataListURL: "/sys/oss/page",
  getDataListIsPage: true,
  deleteURL: "/sys/oss",
  deleteIsBatch: true,
  dataForm: {
    name: '',
    url: '',
  },
  configVisible: false,
  uploadVisible: false
});

const state = reactive({...useView(view), ...toRefs(view)});
baseService.get("/sys/oss/tree", {}).then((res) => {
  treeData.value = res.data
});
// 云存储配置
const configHandle = () => {
  state.configVisible = true;
  nextTick(() => {
    configRef.value.init();
  });
};

// 上传文件
const uploadHandle = () => {
  state.uploadVisible = true;
  nextTick(() => {
    uploadRef.value.init();
  });
};
const copyText = (text: any) => {
  var content = document.getElementById('selectInput');
  if(content == null){
    return;
  }
  selectInput.hidden = false
  content.value = text;
  content.select();
  document.execCommand('copy');
  selectInput.hidden = true;
};
</script>
