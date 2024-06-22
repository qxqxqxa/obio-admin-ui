<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <el-form-item prop="firstName" :label="$t('user.username')">
        <el-input v-model="dataForm.firstName" :placeholder="'First Name'"></el-input>
      </el-form-item>
      <el-form-item prop="lastName" :label="'Last Name'" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.lastName" :placeholder="'Last Name'"></el-input>
      </el-form-item>
      <el-form-item prop="password" :label="$t('user.password')" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.password" type="password" :placeholder="$t('user.password')"></el-input>
      </el-form-item>
      <el-form-item prop="email" :label="$t('user.email')">
        <el-input v-model="dataForm.email" :placeholder="$t('user.email')"></el-input>
      </el-form-item>
      <el-form-item prop="mobile" :label="$t('user.mobile')">
        <el-input v-model="dataForm.mobile" :placeholder="$t('user.mobile')"></el-input>
      </el-form-item>
      <el-form-item prop="roleIdList" :label="$t('user.roleIdList')" class="role-list">
        <el-select v-model="dataForm.roleIdList" multiple :placeholder="$t('user.roleIdList')">
          <el-option v-for="role in roleList" :key="role.id" :label="role.name" :value="role.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="status" :label="$t('user.status')">
        <el-radio-group v-model="dataForm.status">
          <el-radio :value="0">{{ $t("user.status0") }}</el-radio>
          <el-radio :value="1">{{ $t("user.status1") }}</el-radio>
        </el-radio-group>
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
import {isEmail, isMobile} from "@/utils/utils";
import {IObject} from "@/types/interface";
import {useI18n} from "vue-i18n";
import {ElMessage} from "element-plus";

const {t} = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const roleList = ref<any[]>([]);
const postList = ref<any[]>([]);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  firstName: "",
  lastName: "",
  deptId: "",
  password: "",
  gender: 0,
  email: "",
  mobile: "",
  roleIdList: [] as IObject[],
  postIdList: [] as IObject[],
  status: 1
});

const validatePassword = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (!dataForm.id && !/\S/.test(value)) {
    return callback(new Error(t("validate.required")));
  }
  callback();
};
const validateEmail = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (value && !isEmail(value)) {
    return callback(new Error(t("validate.format", {attr: t("user.email")})));
  }
  callback();
};
const validateMobile = (rule: any, value: string, callback: (e?: Error) => any): any => {
  if (value && !isMobile(value)) {
    return callback(new Error(t("validate.format", {attr: t("user.mobile")})));
  }
  callback();
};
const rules = ref({
  firstName: [{required: true, message: t("validate.required"), trigger: "blur"}],
  lastName: [{required: true, message: t("validate.required"), trigger: "blur"}],
  password: [{validator: validatePassword, trigger: "blur"}],
  email: [{required: true, validator: validateEmail, trigger: "blur"}],
  mobile: [{validator: validateMobile, trigger: "blur"}]
});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";
  dataForm.deptId = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  Promise.all([getRoleList()]).then(() => {
    if (id) {
      getInfo(id);
    }
  });
};

// 获取角色列表
const getRoleList = () => {
  return baseService.get("/sys/role/list").then((res) => {
    roleList.value = res.data;
  });
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get(`/sys/user/${id}`).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/user", {
      ...dataForm,
      roleIdList: [...dataForm.roleIdList]
    }).then(() => {
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

<style lang="less" scoped>
.mod-sys__user {
  .role-list {
    .el-select {
      width: 100%;
    }
  }
}
</style>
