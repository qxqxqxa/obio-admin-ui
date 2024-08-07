<template>
  <div style="border: 1px solid #ccc; width: 100%; z-index: 100 ">
    <!-- 工具栏 -->
    <Toolbar :editor="editorRef" :mode="mode" style="border-bottom: 1px solid #ccc" />
    <!-- 编辑器 -->
    <Editor :model-value="modelValue" :style="style" :disabled="disabled" :default-config="editorConfig" :mode="mode" @onCreated="handleCreated" @onChange="handleChange" />
  </div>
</template>

<script lang="ts" setup>
import "@wangeditor/editor/dist/css/style.css";
import { onBeforeUnmount, shallowRef } from "vue";
import { Editor, Toolbar } from "@wangeditor/editor-for-vue";
import { IDomEditor, IEditorConfig } from "@wangeditor/editor";
import app from "@/constants/app";
import { getToken } from "@/utils/cache";

const props = defineProps({
  modelValue: {
    type: String,
    required: true
  },
  mode: {
    type: String,
    default: "default" // 可选值：[default | simple]
  },
  placeholder: {
    type: String,
    default: ""
  },
  style: {
    type: String,
    default: "height: 300px;"
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

// 编辑器实例，必须用 shallowRef
const editorRef = shallowRef();

type InsertFnType = (url: string, alt: string, href: string) => void;

// 编辑器配置
const editorConfig: Partial<IEditorConfig> = {
  placeholder: props.placeholder,
  readOnly: props.disabled,
  MENU_CONF: {
    uploadImage: {
      server: `${app.api}/sys/oss/upload?token=${getToken()}`, // 上传地址
      fieldName: "file",
      // 自定义插入图片
      customInsert(res: any, insertFn: InsertFnType) {
        // res 即服务端的返回结果
        // 从 res 中找到 url alt href ，然后插图图片
        insertFn(res.data.src, "", "");
      }
    },
    fontFamily: {
      fontFamilyList: [
        // 'Arial',
        // 'SegoeUI',
        // 'Tahoma',
        // 'Verdana',
        // { name: '仿宋', value: '仿宋' },
        // 字符串形式
        '黑体',
        '仿宋',
        '楷体',
        '华文仿宋',
        '华文楷体',
        '宋体',
        '微软雅黑',
        'SegoeUI',
        'Arial',
        'Tahoma',
        'Verdana',
        'Times New Roman',
        'Courier New',
      ]
    }
  }
};

// 组件销毁时，也及时销毁编辑器
onBeforeUnmount(() => {
  const editor = editorRef.value;
  if (editor == null) {
    return;
  }
  editor.destroy();
});

const handleCreated = (editor: IDomEditor) => {
  editorRef.value = editor;
};

// 编辑器change事件触发
const emit = defineEmits(["update:modelValue"]);
const handleChange = (editor: IDomEditor) => {
  emit("update:modelValue", editor.getHtml());
};
</script>
