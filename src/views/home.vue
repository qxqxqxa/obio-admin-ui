<template>
  <div class="mod-home">
    <el-row :gutter="20">
      <el-col :span="12" :xs="24">
        <table>
          <tr>
            <th>{{ $t("home.sysInfo.name") }}</th>
            <td>{{ $t("home.sysInfo.nameVal") }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.version") }}</th>
            <td>{{ $t("home.sysInfo.versionVal") }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.osName") }}</th>
            <td>{{ sysInfo.osName }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.osVersion") }}</th>
            <td>{{ sysInfo.osVersion }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.osArch") }}</th>
            <td>{{ sysInfo.osArch }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.processors") }}</th>
            <td>{{ sysInfo.processors }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.totalPhysical") }}</th>
            <td>{{ sysInfo.totalPhysical }}MB</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.freePhysical") }}</th>
            <td>{{ sysInfo.freePhysical }}MB</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.memoryRate") }}</th>
            <td>{{ sysInfo.memoryRate }}%</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.userLanguage") }}</th>
            <td>{{ sysInfo.userLanguage }}</td>
          </tr>
        </table>
      </el-col>
      <el-col :span="12" :xs="24">
        <table>
          <tr>
            <th>{{ $t("home.sysInfo.jvmName") }}</th>
            <td>{{ sysInfo.jvmName }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.javaVersion") }}</th>
            <td>{{ sysInfo.javaVersion }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.javaHome") }}</th>
            <td>{{ sysInfo.javaHome }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.userDir") }}</th>
            <td>{{ sysInfo.userDir }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.javaTotalMemory") }}</th>
            <td>{{ sysInfo.javaTotalMemory }}MB</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.javaFreeMemory") }}</th>
            <td>{{ sysInfo.javaFreeMemory }}MB</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.javaMaxMemory") }}</th>
            <td>{{ sysInfo.javaMaxMemory }}MB</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.userName") }}</th>
            <td>{{ sysInfo.userName }}</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.systemCpuLoad") }}</th>
            <td>{{ sysInfo.systemCpuLoad }}%</td>
          </tr>
          <tr>
            <th>{{ $t("home.sysInfo.userTimezone") }}</th>
            <td>{{ sysInfo.userTimezone }}</td>
          </tr>
        </table>
      </el-col>
    </el-row>
  </div>
</template>

<script lang="ts" setup>
import { onMounted, reactive } from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";

onMounted(() => {
  getSysInfo();
});

const sysInfo = reactive({
  osName: "",
  osVersion: "",
  osArch: "",
  processors: 0,
  totalPhysical: 0,
  freePhysical: 0,
  memoryRate: 0,
  userLanguage: "",
  jvmName: "",
  javaVersion: "",
  javaHome: "",
  userDir: "",
  javaTotalMemory: 0,
  javaFreeMemory: 0,
  javaMaxMemory: 0,
  userName: "",
  systemCpuLoad: 0,
  userTimezone: ""
});

const getSysInfo = () => {
  baseService.get("/sys/info").then((res) => {
    if (res.code !== 0) {
      return ElMessage.error(res.msg);
    }
    Object.assign(sysInfo, res.data);
  });
};
</script>

<style lang="less">
.mod-home {
  table {
    width: 100%;
    border: 1px solid #ebeef5;
    border-collapse: collapse;
    th,
    td {
      padding: 12px 10px;
      border: 1px solid #ebeef5;
    }
    th {
      width: 30%;
    }
  }
}
</style>
