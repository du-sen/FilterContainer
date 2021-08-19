<template>
  <div>
    <!-- 普通表单元素 -->
    <template v-for="(item, index) in filterInfo">
      <div v-if="!item.isAdvanced" :key="item.key" class="filter-item">
        <!-- 表单元素label -->
        <span class="filter-label"> {{ item.label }}： </span>

        <!-- 单个表单元素 -->
        <form-item
          :item="item"
          :value.sync="formData[item.key]"
          @change="onChange(formData, formData[item.key], item.key, index)"
        >
        </form-item>
      </div>
    </template>

    <!-- Form-Slot -->
    <slot :formData="formData" name="formSlot"></slot>

    <!-- 操作区 -->

    <!-- 高级搜索按钮 -->
    <el-button
      v-if="isAdvancedBtn"
      class="filter-item"
      size="small"
      type="text"
      @click="isAdvancedFrom = !isAdvancedFrom"
      >高级搜索</el-button
    >

    <!-- 清空按钮 -->
    <el-button v-if="hasClear" class="filter-item" size="small" @click="onClear"
      >重 置</el-button
    >

    <!-- Operate-Slot -->
    <slot :formData="formData" name="operateSlot"></slot>

    <!-- 高级表单元素 -->
    <transition-group
      class="filter-item-group"
      tag="div"
      name="filter-show-more"
    >
      <template v-for="(item, index) in filterInfo">
        <div
          v-if="item.isAdvanced"
          v-show="
            item.isAdvanced ? isAdvancedFrom : item.config && item.config.show
          "
          :key="item.key"
          class="filter-item"
        >
          <!-- 表单元素label -->
          <span class="filter-label"> {{ item.label }}： </span>

          <!-- 单个表单元素 -->
          <form-item
            :item="item"
            :value.sync="formData[item.key]"
            @change="onChange(formData, formData[item.key], item.key, index)"
          >
          </form-item>
        </div>
      </template>
    </transition-group>
  </div>
</template>

<script>
import FormItem from "./FormItem";
export default {
  name: "FilterForm",
  components: { FormItem },
  props: {
    // filter表单配置项
    dataJson: {
      type: Array,
      default: () => []
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
    }
  },
  data() {
    return {
      nullFilterData: {}, //初始筛选数据对象-用于数据清空
      formData: {}, // 筛选数据对象
      filterInfo: null,
      isAdvancedBtn: false, // 是否显示高级按钮
      isAdvancedFrom: false // 控制显示高级表单元素
    };
  },
  watch: {
    dataJson: {
      handler(val) {
        this.initFilter(val);
      },
      deep: true,
      immediate: true
    },
    filterData: {
      handler(val) {
        // 合并formData, 防止数据丢失
        Object.assign(this.formData, val);
        this.initFilter(this.dataJson);
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    // 数据初始化
    initFilter(dataJson) {
      this.filterInfo = this.deepClone(dataJson);

      this.filterInfo.map(item => {
        if (!item.config) item.config = {};
        // 如果表单元素配置信息为高级, 显示高级按钮
        if (item.isAdvanced) this.isAdvancedBtn = true;

        // 设置默认值
        item.config.size =
          item.config && item.config.size ? item.config.size : "small";
        item.config.show =
          item.config &&
          (item.config.show === true || item.config.show === false)
            ? item.config.show
            : true;

        // 自定义options 的 labelKey、valueKey、nameKey
        if (Array.isArray(item.options))
          item.options = this.getInitOption(item.options, item.optionConfig);

        // 确定初始数据类型, 并添加默认值
        const initData = this.getInitData(item);
        this.$set(this.formData, item.key, initData);
        // 备份一份初始过滤数据, 方便清空
        this.nullFilterData[item.key] = Array.isArray(initData)
          ? []
          : undefined; // 防止引用同一个initData, 产生对象引用问题
      });

      // 重新排序, 高级选项排在最后
      this.filterInfo.sort((a, b) => {
        if (a.isAdvanced === true && !b.isAdvanced) {
          return 1;
        } else if (b.isAdvanced === true && !a.isAdvanced) {
          return -1;
        } else {
          return 0;
        }
      });
    },
    // 自定义options 的 labelKey、valueKey、nameKey
    getInitOption(options, optionConfig) {
      let optionsCache = this.deepClone(options);

      optionsCache.map((option, index, arr) => {
        const isObject =
          Object.prototype.toString.call(option) === "[object Object]";

        if (
          isObject &&
          optionConfig &&
          (optionConfig.labelKey ||
            optionConfig.valueKey ||
            optionConfig.nameKey)
        ) {
          // 表单元素 option 子元素为对象
          const { labelKey, valueKey, nameKey } = optionConfig;
          if (labelKey) option.label = option[labelKey];
          if (valueKey) option.value = option[valueKey];
          if (nameKey) option.name = option[nameKey];
        } else if (!isObject) {
          // 表单元素 option 子元素非对象
          const obj = {
            label: option,
            value: option,
            name: option
          };
          arr[index] = obj;
        }
      });
      return optionsCache;
    },
    // 确定初始数据类型, 并添加默认值
    getInitData(item) {
      let rst;
      const defaultValue = this.formData[item.key];
      const isArray =
        (item.type === "el-checkbox-group" && Array.isArray(item.options)) ||
        item.type === "el-cascader" ||
        item.type === "el-time-select" ||
        (item.config && item.config.type === "datetimerange") ||
        (item.config && item.config.type === "daterange") ||
        (item.config && item.config.type === "monthrange") ||
        (item.config && item.config.type === "dates") ||
        (item.config && item.config.multiple === true);

      // 初始化数据类型, 并添加默认值
      if (isArray) {
        if (Array.isArray(defaultValue)) {
          rst = [...defaultValue];
        } else {
          rst =
            defaultValue === undefined || defaultValue === null
              ? []
              : [defaultValue];
        }
      } else if (!isArray) {
        rst = defaultValue;
      }
      return rst;
    },
    // 重置搜索条件
    onClear() {
      this.formData = { ...this.nullFilterData };
      this.$emit("filterClear", { ...this.nullFilterData });
    },
    // change事件
    onChange(formData, value, key, index) {
      // 更新父组件 filterData
      // this.$emit("update:filterData", formData);
      this.$emit("change", formData, value, key, index);
    },
    // 深拷贝
    deepClone(obj, cache = []) {
      if (obj === null || typeof obj !== "object") {
        return obj;
      }
      const hit = cache.filter(c => c.original === obj)[0];
      if (hit) {
        return hit.copy;
      }
      const copy = Array.isArray(obj) ? [] : {};
      cache.push({
        original: obj,
        copy
      });
      Object.keys(obj).forEach(key => {
        copy[key] = this.deepClone(obj[key], cache);
      });
      return copy;
    }
  }
};
</script>