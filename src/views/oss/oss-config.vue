<template>
  <el-dialog v-model="visible" :title="$t('oss.config')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="140px">
      <el-form-item prop="localDomain" :label="$t('oss.localDomain')">
        <el-input v-model="dataForm.localDomain" :placeholder="$t('oss.localDomainTips')"></el-input>
      </el-form-item>
      <el-form-item prop="localPrefix" :label="$t('oss.localPrefix')">
        <el-input v-model="dataForm.localPrefix" :placeholder="$t('oss.localPrefixTips')"></el-input>
      </el-form-item>
      <el-form-item prop="localPath" :label="$t('oss.localPath')">
        <el-input v-model="dataForm.localPath" :placeholder="$t('oss.localPathTips')"></el-input>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import {reactive, ref} from "vue";
import baseService from "@/service/baseService";
import {useI18n} from "vue-i18n";
import {ElMessage} from "element-plus";

const {t} = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  type: 5,
  localDomain: "",
  localPrefix: "",
  localPath: "",
});

const rules = ref({
  localDomain: [{ required: true, message: t("validate.required"), trigger: "blur" }],
  localPath: [{ required: true, message: t("validate.required"), trigger: "blur" }],
});

const init = () => {
  visible.value = true;

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  getInfo();
};

// 获取信息
const getInfo = () => {
  baseService.get("/sys/oss/info").then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    baseService.post("/sys/oss", dataForm).then(() => {
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

defineExpose({
  init
});
</script>
