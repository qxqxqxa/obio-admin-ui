<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" label-width="120px">
      <el-form-item prop="title" :label="$t('news.title')">
        <el-input v-model="dataForm.title" :placeholder="$t('news.title')"></el-input>
      </el-form-item>
      <el-form-item prop="categoryId" :label="$t('service.category')">
        <el-tree-select v-model="dataForm.categoryId" ref="articlesTree" :data="categories" :props="{ label: 'name', children: 'children' }" :render-after-expand="false" accordion @current-change="treeCurrentChangeHandle" :expand-on-click-node="true" node-key="id" :highlight-current="true"/>
      </el-form-item>
      <el-form-item label="image">
        <el-upload :action="url" :before-upload="beforeUploadHandle" :on-success="successHandle" class="text-center">
          <img v-if="!dataForm.subPic"   style="width: 100px; height: 100px" :src="defaultImg" alt="默认图片">  
         <img v-else :src="dataForm.subPic"  style="width: 100px; height: 100px" alt="图片">  
        </el-upload>
      </el-form-item>
      <el-form-item :label="$t('service.keywords')">
        <el-input v-model="dataForm.keywords" :placeholder="$t('service.keywords')"></el-input>
      </el-form-item>
      <el-form-item prop="summary" :label="$t('service.summary')">
        <el-input type="textarea" :rows="3" v-model="dataForm.summary" :placeholder="$t('service.summary')"></el-input>
      </el-form-item>
      <el-form-item prop="content" :label="$t('service.content')">
        <WangEditor v-model="dataForm.content"></WangEditor>
      </el-form-item>
      <el-form-item prop="sort":label="$t('service.sort')">
        <el-input-number v-model="dataForm.sort" controls-position="right" :min="0" :label="$t('menu.sort')"></el-input-number>
      </el-form-item>
      <el-form-item prop="pubDate" label="Publish Time">
        <el-date-picker v-model="dataForm.pubDate" type="datetime" format="YYYY-MM-DD HH:mm:ss" value-format="YYYY-MM-DD HH:mm:ss"></el-date-picker>
      </el-form-item>
<!--      <el-row>-->
<!--        <el-col  span="12">-->
<!--          <el-form-item prop="pubDate" label="Publish Time">-->
<!--            <el-date-picker v-model="dataForm.pubDate" type="datetime" format="YYYY-MM-DD HH:mm:ss" value-format="YYYY-MM-DD HH:mm:ss"></el-date-picker>-->
<!--          </el-form-item>-->
<!--        </el-col>-->
<!--        <el-col  span="12">-->
<!--          <el-form-item  :label="$t('service.creator')">-->
<!--            <el-input v-model="dataForm.author"></el-input>-->
<!--          </el-form-item>-->
<!--        </el-col>-->
<!--      </el-row>-->

    </el-form>
    <template v-slot:footer>
      <el-button type="primary" @click="dataFormSubmitHandle(1)">Save</el-button>
      <el-button type="primary" @click="saveDraft()">Save Draft</el-button>
      <!--      <el-button type="primary" @click="preView()">PreView</el-button>-->
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import {reactive, ref} from "vue";
import {useAppStore} from "@/store";

import baseService from "@/service/baseService";
import WangEditor from "@/components/wang-editor/index.vue";

import {useI18n} from "vue-i18n";
import {ElMessage} from "element-plus";
import FileUpload from "@/views/oss/file-upload.vue";
import app from "@/constants/app";
import {getToken} from "@/utils/cache";
import {IObject} from "@/types/interface";
import defaultImg from "@/assets/images/default.png";


const {t} = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();
const categories = ref([]);
const dataForm = reactive({
  id: "",
  title: "",
  categoryId: '',
  categoryName: "",
  subPic: "",
  keywords: "",
  summary: "",
  content: "",
  pubDate: "",
  status: 0,
  sort: 0,
});
const url = ref("");
const rules = ref({
  title: [{required: true, message: t("validate.required"), trigger: "blur"}],
  categoryId: [{required: true, message: t("validate.required"), trigger: "change"}],
  content: [{required: true, message: t("validate.required"), trigger: "blur"}],
  eventTime: [{required: true, message: t("validate.required"), trigger: "blur"}],
  pubDate: [{required: true, message: t("validate.required"), trigger: "blur"}],
});

const init =  (id?: number) => {
  visible.value = true;
  dataForm.id = "";
  url.value = `${app.api}/sys/oss/upload?token=${getToken()}`;
  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  Object.assign(dataForm, {
    eventTime: (new Date()).Format("yyyy-MM-dd HH:mm:ss"),
    pubDate: (new Date()).Format("yyyy-MM-dd HH:mm:ss"),
    author: useAppStore().state.user.username,
  });
  getCategoryList();
  if (id) {
    getInfo(id);
  }
};

const getCategoryList = (()=>{
  baseService.get("/cms/categories/public/list?pid=1804037380862840833").then((res) => {
    categories.value = res.data;
  });
});
const getInfo = (id: number) => {
  baseService.get(`/cms/service/${id}`).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = (status: number) => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    dataForm.status = status || 1;
    (!dataForm.id ? baseService.post : baseService.put)("/cms/service", dataForm).then(() => {
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};
// 表单提交
const saveDraft = () => {
  dataFormSubmitHandle(2)
};
// 预览功能
const preView = () => {
  ElMessage.success({message: "todo 预览功能"});
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    return false;
    (!dataForm.id ? baseService.post : baseService.put)("/cms/service", dataForm).then(() => {
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};
// 上传之前
const beforeUploadHandle = (file: IObject) => {
  if (file.type !== "image/jpg" && file.type !== "image/jpeg" && file.type !== "image/png" && file.type !== "image/gif") {
    ElMessage.error(t("upload.tip", {format: "jpg、png、gif"}));
    return false;
  }
};

// 上传成功
const successHandle = (res: IObject, file: IObject, list: IObject[]) => {
  if (res.code !== 0) {
    return ElMessage.error(res.msg);
  }
  dataForm.subPic = res.data.src
  ElMessage.success({
    message: t("prompt.success"),
    duration: 500,
    onClose: () => {
    }
  });
};
const treeCurrentChangeHandle = (data: IObject) => {
  dataForm.categoryId = data.id;
  dataForm.categoryName = data.name;
}
const categoriesTreeCurrentChangeHandle = () => {

};
defineExpose({
  init
});
</script>
