<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('Detail')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm"  :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item label="FirstName"  prop="firstName">
        <el-input v-model="dataForm.firstName" placeholder="" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="LastName" prop="lastName">
        <el-input v-model="dataForm.lastName" placeholder="" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="Company" prop="company">
        <el-input v-model="dataForm.company" placeholder="Company" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="Email" prop="email">
        <el-input v-model="dataForm.email" placeholder="Email" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="Phone" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="Phone" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="InquiredItem" prop="inquiredItem">
        <el-input type="textarea" v-model="dataForm.inquiredItem" placeholder="InquiredItem" :readonly="dataForm.id"></el-input>
      </el-form-item>
      <el-form-item label="Comment" prop="comment">
        <el-input  type="textarea" v-model="dataForm.comment" placeholder="Comment" :readonly="dataForm.id"></el-input>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <!-- <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button> -->
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  firstName: "",
  lastName: "",
  company: "",
  email: "",
  phone: "",
  inquiredItem: "",
  comment: "",
  fromEmail: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
});

const rules = ref({
});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if (id) {
    getInfo(id);
  }
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/cms/inquiry/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/cms/inquiry/save", dataForm).then((res) => {
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