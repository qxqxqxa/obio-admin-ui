<script lang="ts">
import SvgIcon from "@/components/base/svg-icon";
import { EThemeSetting } from "@/constants/enum";
import { getThemeConfigCache, setThemeConfigToCache } from "@/utils/theme";
import { copyToClipboard } from "@/utils/utils";
import { ElMessage } from "element-plus";
import { defineComponent, reactive } from "vue";
import { useI18n } from "vue-i18n";
import SettingNavLayout from "./setting-nav-layout.vue";
import SettingOther from "./setting-other.vue";
import SettingTheme from "./setting-theme.vue";
import "@/assets/css/setting.less";

/**
 * 可视化设置窗口
 */
export default defineComponent({
  name: "SettingSidebar",
  components: { SettingNavLayout, SettingOther, SettingTheme, SvgIcon },
  setup() {
    const { t } = useI18n();
    const themeConfigCache = getThemeConfigCache();
    const state = reactive({ openDrawer: false, config: themeConfigCache });

    const onOpenSettingThemeWindow = () => {
      state.openDrawer = true;
    };
    const onChange = (key: EThemeSetting, value: string | boolean | number) => {
      setThemeConfigToCache(key, value);
    };
    const onCopySetting = () => {
      const vl = getThemeConfigCache();
      copyToClipboard(JSON.stringify({ ...vl, "//": t("ui.setting.settingTips") }, null, 2));
      ElMessage.success(t("ui.setting.copySuccess"));
    };
    return { state, onOpenSettingThemeWindow, onChange, onCopySetting, t };
  }
});
</script>
