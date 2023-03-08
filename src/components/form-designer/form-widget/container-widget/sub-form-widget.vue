<template>
  <container-wrapper :widget="widget" :designer="designer" :parent-widget="parentWidget" :parent-list="parentList"
                     :index-of-parent-list="indexOfParentList">

    <div :key="widget.id" class="sub-form-container"
         :class="{'selected': selected}" @click.stop="selectWidget(widget)">
      <draggable :list="widget.widgetList" v-bind="{group:'dragGroup', ghostClass: 'ghost',animation: 200}"
                 handle=".drag-handler"
                 @add="(evt) => onContainerDragAdd(evt, widget.widgetList)"
                 @update="onContainerDragUpdate" :move="checkContainerMove">
        <transition-group name="fade" tag="div" class="form-widget-list">
          <template v-for="(subWidget, swIdx) in widget.widgetList">

            <template v-if="'container' === subWidget.category">
              <component
                :is="subWidget.type + '-widget'"
                :widget="subWidget"
                :designer="designer"
                :key="subWidget.id"
                :parent-list="widget.widgetList"
                :index-of-parent-list="swIdx"
                :parent-widget="widget"/>
            </template>
            <template v-else>
              <div
                class="item"
                :style="{width: subWidget.options.columnWidth}"
                :key="subWidget.id+'row'"
              >
                <component
                  :is="subWidget.type + '-widget'"
                  :field="subWidget"
                  :designer="designer"
                  :key="subWidget.id"
                  :parent-list="widget.widgetList"
                  :index-of-parent-list="swIdx"
                  :parent-widget="widget"
                  :design-state="true"
                />
              </div>
            </template>
          </template>
        </transition-group>
      </draggable>
    </div>

  </container-wrapper>
</template>

<script>
  import emitter from '@/utils/emitter'
  import i18n from "@/utils/i18n"
  import FieldComponents from '@/components/form-designer/form-widget/field-widget/index'
  import ContainerWrapper from "@/components/form-designer/form-widget/container-widget/container-wrapper.vue";
  import containerMixin from "@/components/form-designer/form-widget/container-widget/containerMixin.js";
  import refMixinDesign from "@/components/form-designer/refMixinDesign.js";
  import Draggable from "vuedraggable";

  export default {
    name: "sub-form-widget",
    componentName: 'ContainerWidget',
    mixins: [emitter, i18n, refMixinDesign, containerMixin],
    components: {
      Draggable,
      ContainerWrapper,
      ...FieldComponents,
    },
    props: {
      widget: Object,
      parentWidget: Object,
      parentList: Array,
      indexOfParentList: Number,
      designer: Object,
    },
    inject: ['refList'],
    data() {
      return {
        rowIdData: [],
        fieldSchemaData: [],
        actionDisabled: false,
      }
    },
    computed: {
      selected() {
        return this.widget.id === this.designer.selectedId
      },

      customClass() {
        return this.widget.options.customClass || ''
      },

    },
    created() {
      this.initRefList()
    },
    mounted() {
      //默认添加首行后，主动触发相关事件！！
    },
    methods: {}
  }
</script>

<style lang="scss" scoped>
  .sub-form-container {
    padding: 8px;
    border: 1px dashed hsla(0,0%,66.7%,.75);
    .item{
      display: inline-block;
      &::v-deep{
        .el-form-item__content{
          margin-left: 0 !important;
        }
      }
      &+.item{
        margin-left: 8px;
      }
    }
    &::v-deep{
      .el-form-item{
        margin-bottom: 0;
      }
      .ghost{
        height: 100%;
        width: 2px;
        min-height: 50px;
        display: inline-block;
      }
    }

  }
  .sub-form-container.selected {
    outline: 2px solid $--color-primary !important;
  }
  .sub-form-container .form-widget-list{
    min-height: 50px;
  }
</style>
