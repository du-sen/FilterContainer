<template>
  <el-row class="filter-container clearfix">
    <el-col class="header-left" :span="span[0]">
      <h3 class="table-caption">{{ titleInfo.title }}</h3>
      <el-tooltip
        v-if="titleInfo.summary"
        class="table-caption-tip"
        effect="dark"
        :content="titleInfo.summary"
        placement="top"
      >
        <i class="el-icon-question"></i>
      </el-tooltip>
      <slot name="titleSlot"></slot>
    </el-col>
    <el-col :span="span[1]" class="header-right">
      <filter-form
        :data-json="dataJson"
        :filter-data="filterData"
        :has-clear="hasClear"
        @change="onChange"
        @filterClear="onClear"
      >
        <!-- 筛选功能-表单元素slot -->
        <template v-slot:formSlot="slotProps">
          <slot :filterData="slotProps.formData" name="formSlot"></slot>
        </template>

        <!-- 操作功能-slot -->
        <template v-slot:operateSlot="slotProps">
          <slot :filterData="slotProps.formData" name="operateSlot"></slot>
        </template>
      </filter-form>
    </el-col>
  </el-row>
</template>

<script>
import FilterForm from "./FilterForm";

export default {
  name: "FilterContainer",
  components: {
    FilterForm
  },
  props: {
    // filter表单配置项
    dataJson: {
      type: Array,
      default: () => []
    },
    // 标题配置项
    titleInfo: {
      type: Object,
      default: () => ({
        title: "",
        summary: ""
      })
    },
    // filter表单数据
    filterData: {
      type: Object,
      default: () => {}
    },
    // 是否显示clear按钮
    hasClear: {
      type: Boolean,
      default: false
    },
    // 控制组件栅格布局
    span: {
      type: Array,
      default: () => [3, 21]
    }
  },
  data() {
    return {};
  },
  methods: {
    onChange(formData, value, key, index) {
      // 更新父组件 filterData
      this.$emit("update:filterData", this.jsonClone(formData));
      this.$emit("change", this.jsonClone(formData), value, key, index);
    },
    onClear(nullFilterData) {
      // 更新父组件 filterData
      this.$emit("update:filterData", this.jsonClone(nullFilterData));
      this.$emit("filterClear", this.jsonClone(nullFilterData));
    },
    jsonClone(data) {
      return JSON.parse(JSON.stringify(data));
    }
  }
};
</script>