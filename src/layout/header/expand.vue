<script lang="ts">
import SvgIcon from "@/components/base/svg-icon";
import app from "@/constants/app";
import baseService from "@/service/baseService";
import { getToken } from "@/utils/cache";
import { checkPermission } from "@/utils/utils";
import { useFullscreen, useWebSocket } from "@vueuse/core";
import { ElNotification } from "element-plus";
import { computed, defineComponent, ref, watch } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
import { useAppStore } from "@/store";
import SettingSidebar from "../setting/index.vue";
import userLogo from "@/assets/images/user.png";
import "@/assets/css/header.less";
import { ElMessageBox } from "element-plus";

interface IExpand {
  userName?: string;
}

/**
 * 顶部右侧扩展区域
 */
export default defineComponent({
  name: "Expand",
  components: { SettingSidebar, SvgIcon },
  props: {
    userName: String
  },
  setup(props: IExpand) {
    const { t } = useI18n();
    const router = useRouter();
    const store = useAppStore();
    const { isFullscreen, toggle } = useFullscreen();
    const errorNotificationCount = ref(0);

    watch(
      () => store.state.appIsLogin,
      (vl) => {
        if (vl) {
          // ws();
        }
      }
    );

    const ws = () => {
      const { data } = useWebSocket(app.websocket + "?token=" + getToken(), {
        autoReconnect: true,
        heartbeat: {
          interval: 15000
        },
        onConnected() {
          console.log("websocket connected");
        },
        onError() {
          errorNotificationCount.value++;
          if (errorNotificationCount.value < 2) {
            ElNotification({
              type: "error",
              title: t("notice.disconnect"),
              message: t("notice.disconnectMessage")
            });
            console.error("websocket connect error");
          }
        }
      });
      watch(
        () => data.value,
        (vl) => {
          const result = JSON.parse(vl);
          // 如果是有新文本通知，则提示有新通知
          if (result.type === 0) {
            ElNotification({
              title: t("notice.new"),
              message: result.msg,
              type: "info",
              duration: 5000
            });
          }
        }
      );
    };

    const onClickUserMenus = (path: string) => {
      if (path === "/login") {
        ElMessageBox.confirm(t("prompt.info", { handle: t("logout") }), t("prompt.title"), {
          confirmButtonText: t("confirm"),
          cancelButtonText: t("cancel"),
          type: "warning"
        })
          .then(() => {
            baseService.post("/logout").finally(() => {
              router.push(path);
            });
          })
          .catch(() => {
            //
          });
      } else {
        router.push(path);
      }
    };
    const onClickMessage = () => {
      router.push("/notice/notice-user");
    };
    const messagePermission = computed(() => checkPermission(store.state.permissions, "sys:notice:all"));
    return {
      props,
      store,
      isFullscreen,
      messagePermission,
      userLogo,
      onClickUserMenus,
      onClickMessage,
      toggle,
      t
    };
  }
});
</script>
<template>
  <div class="rr-header-right-items">
    <div @click="toggle" class="hidden-xs-only">
      <span>
        <svg-icon :name="isFullscreen ? 'tuichuquanping' : 'fullscreen2'"></svg-icon>
      </span>
    </div>
    <div style="display: flex; justify-content: center; align-items: center">
      <el-dropdown @command="onClickUserMenus">
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item icon="lock" command="/user/password">
              {{ t("ui.user.links.editPassword") }}
            </el-dropdown-item>
            <el-dropdown-item icon="switch-button" divided command="/login">
              {{ t("ui.user.links.logout") }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
        <span class="el-dropdown-link" style="display: flex">
          {{ props.userName }}
          <el-icon class="el-icon--right" style="font-size: 14px"><arrow-down /></el-icon>
        </span>
      </el-dropdown>
    </div>
    <setting-sidebar></setting-sidebar>
  </div>
</template>
