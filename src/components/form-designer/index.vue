<!--
/**
 * author: vformAdmin
 * email: vdpadmin@163.com
 * website: https://www.vform666.com
 * date: 2021.08.18
 * remark: 如果要分发VForm源码，需在本文件顶部保留此文件头信息！！
 */
-->

<template>
  <el-container class="main-container full-height">
    <el-header class="main-header">
      <div class="float-left main-title">
        <span class="bold">VForm</span> {{ i18nt('application.productTitle') }} <span
        class="version-span">Ver {{ vFormVersion }}</span></div>
      <div class="float-right external-link">
        <el-dropdown :hide-timeout="2000" @command="handleLanguageChanged">
          <span class="el-dropdown-link">{{ curLangName }}<i class="el-icon-arrow-down el-icon--right"></i></span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="zh-CN">{{ i18nt('application.zh-CN') }}</el-dropdown-item>
            <el-dropdown-item command="en-US">{{ i18nt('application.en-US') }}</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </el-header>

    <el-container>
      <el-aside class="side-panel">
        <widget-panel :designer="designer"/>
      </el-aside>

      <el-container class="center-layout-container">
        <el-header class="toolbar-header">
          <toolbar-panel :designer="designer" :global-dsv="globalDsv" ref="toolbarRef">
            <template v-for="(idx, slotName) in $slots" #[slotName]>
              <slot :name="slotName"></slot>
            </template>
          </toolbar-panel>
        </el-header>
        <el-main class="form-widget-main">
          <el-scrollbar class="container-scroll-bar" :style="{height: scrollerHeight}">
            <v-form-widget :designer="designer" :form-config="designer.formConfig" :global-dsv="globalDsv"
                           ref="formRef">
            </v-form-widget>
          </el-scrollbar>
        </el-main>
      </el-container>

      <el-aside>
        <setting-panel :designer="designer" :selected-widget="designer.selectedWidget"
                       :form-config="designer.formConfig" :global-dsv="globalDsv"/>
      </el-aside>
    </el-container>

  </el-container>
</template>

<script>
import WidgetPanel from './widget-panel/index'
import ToolbarPanel from './toolbar-panel/index'
import SettingPanel from './setting-panel/index'
import VFormWidget from './form-widget/index'
import {createDesigner} from "@/components/form-designer/designer"
import {
  addWindowResizeHandler
} from "@/utils/util"
import {VARIANT_FORM_VERSION} from "@/utils/config"
import i18n, {changeLocale} from "@/utils/i18n"
import axios from "axios"


const defaultDesignerConfig = {
  //是否显示语言切换菜单
  languageMenu: true,
  //是否显示表单模板
  formTemplates: true,
  //是否显示组件事件属性折叠面板
  eventCollapse: true,
  //禁止修改组件名称
  widgetNameReadonly: false,
  //是否显示清空设计器按钮
  clearDesignerButton: true,
  //是否显示预览表单按钮
  previewFormButton: true,
  //是否显示导入JSON按钮
  importJsonButton: true,
  //是否显示导出JSON器按钮
  exportJsonButton: true,
  //是否显示导出代码按钮
  exportCodeButton: true,
  //是否显示生成SFC按钮
  generateSFCButton: true,
  //设计器工具按钮栏最大宽度（单位像素）
  toolbarMaxWidth: 420,
  //设计器工具按钮栏最小宽度（单位像素）
  toolbarMinWidth: 300,
  //设计器预设CSS样式代码
  presetCssCode: '',
  //是否在设计器初始化时将表单内容重置为空
  resetFormJson: false
}


export default {
  name: "VFormDesigner",
  componentName: "VFormDesigner",
  mixins: [i18n],
  components: {
    WidgetPanel,
    ToolbarPanel,
    SettingPanel,
    VFormWidget
  },
  props: {
    /* 后端字段列表API */
    fieldListApi: {
      type: Object,
      default: null,
    },

    /* 禁止显示的组件名称数组 */
    bannedWidgets: {
      type: Array,
      default: () => []
    },

    designerConfig: {
      type: Object,
      default: () => {
        return defaultDesignerConfig
      }
    },

    /* 全局数据源变量 */
    globalDsv: {
      type: Object,
      default: () => ({})
    },

  },
  data() {
    return {
      vFormVersion: VARIANT_FORM_VERSION,
      curLangName: '',
      scrollerHeight: 0,

      designer: createDesigner(this),

      fieldList: []
    }
  },
  provide() {
    return {
      serverFieldList: this.fieldList,
      getDesignerConfig: () => Object.assign(defaultDesignerConfig, this.designerConfig),
      getBannedWidgets: () => this.bannedWidgets
    }
  },
  created() {
  },
  mounted() {
    this.initLocale()

    this.scrollerHeight = window.innerHeight - 56 - 36 + 'px'
    addWindowResizeHandler(() => {
      this.$nextTick(() => {
        this.scrollerHeight = window.innerHeight - 56 - 36 + 'px'
      })
    })


    this.loadFieldListFromServer()
  },
  methods: {
    initLocale() {
      const curLocale = localStorage.getItem('v_form_locale') || 'zh-CN'
      this.curLangName = this.i18nt('application.' + curLocale)
      this.changeLanguage(curLocale)
    },

    loadFieldListFromServer() {
      if (!this.fieldListApi) {
        return
      }

      let headers = this.fieldListApi.headers || {}
      axios.get(this.fieldListApi.URL, {'headers': headers}).then(res => {
        let labelKey = this.fieldListApi.labelKey || 'label'
        let nameKey = this.fieldListApi.nameKey || 'name'

        this.fieldList.splice(0, this.fieldList.length)  //清空已有
        res.data.forEach(fieldItem => {
          this.fieldList.push({
            label: fieldItem[labelKey],
            name: fieldItem[nameKey]
          })
        })
      }).catch(error => {
        this.$message.error(error)
      })
    },

    handleLanguageChanged(command) {
      this.changeLanguage(command)
      this.curLangName = this.i18nt('application.' + command)
    },

    changeLanguage(langName) {
      changeLocale(langName)
    },

    clearDesigner() {
      this.$refs.toolbarRef.clearFormWidget()
    },

    /**
     * 预览表单
     */
    previewForm() {
      this.$refs.toolbarRef.previewForm()
    },

    /**
     * 导入表单JSON
     */
    importJson() {
      this.$refs.toolbarRef.importJson()
    },

    /**
     * 导出表单JSON
     */
    exportJson() {
      this.$refs.toolbarRef.exportJson()
    },

    /**
     * 导出Vue/HTML代码
     */
    exportCode() {
      this.$refs.toolbarRef.exportCode()
    },

    //TODO: 增加更多方法！！

  }
}
</script>

<style lang="scss" scoped>
.el-container.main-container {
  background: #fff;

  ::v-deep aside { /* 防止aside样式被外部样式覆盖！！ */
    margin: 0;
    padding: 0;
    background: inherit;
  }
}

.el-container.full-height {
  height: 100%;
  overflow-y: hidden;
}

.el-container.center-layout-container {
  min-width: 680px;
  border-left: 2px dotted #EBEEF5;
  border-right: 2px dotted #EBEEF5;
}

.el-header.main-header {
  border-bottom: 2px dotted #EBEEF5;
  height: 48px !important;
  line-height: 48px !important;
  min-width: 800px;
}

div.main-title {
  font-size: 18px;
  color: #242424;
  display: flex;
  align-items: center;
  justify-items: center;

  img {
    cursor: pointer;
    width: 36px;
    height: 36px;
  }

  span.bold {
    font-size: 20px;
    font-weight: bold;
    margin: 0 6px 0 6px;
  }

  span.version-span {
    font-size: 14px;
    color: #101F1C;
    margin-left: 6px;
  }
}

.float-left {
  float: left;
}

.float-right {
  float: right;
}

.el-dropdown-link {
  margin-right: 12px;
  cursor: pointer;
}

div.external-link a {
  font-size: 13px;
  text-decoration: none;
  margin-right: 10px;
  color: #606266;
}

.el-header.toolbar-header {
  font-size: 14px;
  border-bottom: 1px dotted #CCCCCC;
  height: 42px !important;
  //line-height: 42px !important;
}

.el-aside.side-panel {
  width: 260px !important;
  overflow-y: hidden;
}

.el-main.form-widget-main {
  padding: 0;

  position: relative;
  overflow-x: hidden;
}

.container-scroll-bar {
  ::v-deep .el-scrollbar__wrap, ::v-deep .el-scrollbar__view {
    overflow-x: hidden;
  }
}
</style>
