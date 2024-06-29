<template>
  <el-form :model="dataForm" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
    <el-form-item prop="name" :label="$t('configInfo.name')">
      <el-input v-model="dataForm.name"></el-input>
    </el-form-item>
    <el-form-item prop="copyright" :label="$t('configInfo.copyright')">
      <el-input v-model="dataForm.copyright"></el-input>
    </el-form-item>
    <el-form-item prop="logo" :label="$t('configInfo.logo')">
      <el-upload class="avatar-uploader" :action="url" :show-file-list="false" :on-success="uploadSuccessHandle">
        <img v-if="dataForm.logo" :src="dataForm.logo" class="avatar" style="width: 100px;"/>
        <el-icon v-else class="avatar-uploader-icon">
          <Plus/>
        </el-icon>
      </el-upload>
    </el-form-item>
    <el-form-item prop="introduce" :label="$t('configInfo.introduce')">
      <el-input v-model="dataForm.introduce"></el-input>
    </el-form-item>
    <el-form-item prop="remark" :label="$t('configInfo.remark')">
      <el-input v-model="dataForm.remark"></el-input>
    </el-form-item>
    <el-form-item :label="$t('configInfo.lan')">
      <el-select v-model="dataForm.lan" size="small">
        <el-option :key="1" label="English" value="en-US"></el-option>
        <el-option :key="1" label="中文" value="zh-CN"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item prop="remark" label="">
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ t("confirm") }}</el-button>
    </el-form-item>
  </el-form>
</template>

<script lang="ts" setup>
import {computed, nextTick, reactive, ref} from "vue";
import baseService from "@/service/baseService";
import {IObject} from "@/types/interface";
import {useAppStore} from "@/store";
import {useI18n} from "vue-i18n";
import {ElMessage} from "element-plus";
import app from "@/constants/app";
import {getToken, setCache} from "@/utils/cache";
import {CacheLang} from "@/constants/cacheKey";

const store = useAppStore();
const {t} = useI18n();
const url = ref("");
const emit = defineEmits(["refreshDataList"]);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name: "",
  copyright: "",
  logo: "",
  introduce: "",
  pub1: "",
  pub2: "",
  pub3: "",
  remark: "",
  lan: "",
});

const user = computed(() => store.state.user);
// 上传图片成功
const uploadSuccessHandle = (res: IObject) => {
   dataForm.logo = res.data.src;
};

// 获取信息
const getInfo = () => {
  url.value = `${app.api}/sys/oss/upload?token=${getToken()}`;

  baseService.get(`/sys/config/info`).then((res) => {
    Object.assign(dataForm, res.data);
    Object.assign(store.state.configInfo, res.data);
    setCache(CacheLang, res.data.lan);
    (document.querySelector('link[rel="icon"]') || {}).href = res.data.logo;
    (document.querySelector('title') || {innerText: 0}).innerText = res.data.name
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.put("/sys/config", dataForm).then((res) => {
      getInfo();
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          emit("refreshDataList");
        }
      });
    });
  });
};
getInfo();
</script>

<style lang="less" scoped>
.mod-sys__dept {
  .dept-list {
    .el-input__inner,
    .el-input__suffix {
      cursor: pointer;
    }
  }
}
</style>
