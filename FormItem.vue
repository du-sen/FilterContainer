<template>
  <div>
    <!-- Radio -->
    <el-radio-group
      v-if="item.type === 'el-radio-group' && item.options"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
      <template v-if="item.optionType === 'button'">
        <el-radio-button
          v-for="option in item.options"
          :key="option.label"
          v-bind="option"
          >{{ option.name }}</el-radio-button
        >
      </template>

      <template v-else>
        <el-radio
          v-for="option in item.options"
          :key="option.label"
          v-bind="option"
          >{{ option.name }}</el-radio
        >
      </template>
    </el-radio-group>

    <!-- Checkbox -->
    <el-checkbox-group
      v-else-if="item.type === 'el-checkbox-group' && item.options"
      v-model="itemValue"
      v-bind="item.config"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      @change="onChange"
    >
      <template v-if="item.optionType === 'button'">
        <el-checkbox-button
          v-for="option in item.options"
          :key="option.label"
          v-bind="option"
          >{{ option.name }}</el-checkbox-button
        >
      </template>

      <template v-else>
        <el-checkbox
          v-for="option in item.options"
          :key="option.label"
          v-bind="option"
          >{{ option.name }}</el-checkbox
        >
      </template>
    </el-checkbox-group>
    <!-- 无options -->
    <template v-else-if="item.type === 'el-checkbox-group' && !item.options">
      <template v-if="item.optionType === 'button'">
        <el-checkbox-button
          v-model="itemValue"
          :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
          v-bind="item.config"
          @change="onChange"
        ></el-checkbox-button>
      </template>
      <template v-else>
        <el-checkbox
          v-model="itemValue"
          :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
          v-bind="item.config"
          @change="onChange"
        ></el-checkbox>
      </template>
    </template>

    <!-- Input -->
    <el-input
      v-else-if="item.type === 'el-input'"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    ></el-input>

    <!-- InputNumber -->
    <el-input-number
      v-else-if="item.type === 'el-input-number'"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    ></el-input-number>

    <!-- Select -->
    <el-select
      v-else-if="item.type === 'el-select'"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
      <div v-for="group in item.options" :key="group.value">
        <template v-if="!group.options">
          <el-option v-bind="group"></el-option>
        </template>
        <template v-else>
          <el-option-group v-bind="group">
            <el-option
              v-for="option in group.options"
              :key="option.value"
              v-bind="option"
            >
            </el-option>
          </el-option-group>
        </template>
      </div>
    </el-select>

    <!-- Cascader -->
    <el-cascader
      v-else-if="item.type === 'el-cascader'"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
    </el-cascader>

    <!-- Switch -->
    <el-switch
      v-else-if="item.type === 'el-switch'"
      v-model="itemValue"
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
    </el-switch>

    <!-- TimePicker -->
    <el-time-select
      v-else-if="item.type === 'el-time-select'"
      v-model="itemValue"
      :value-format="
        item.config['value-format'] ? item.config['value-format'] : 'timestamp'
      "
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
    </el-time-select>

    <!-- DatePicker -->
    <el-date-picker
      v-else-if="item.type === 'el-date-picker'"
      v-model="itemValue"
      :value-format="
        item.config['value-format'] ? item.config['value-format'] : 'timestamp'
      "
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
    </el-date-picker>

    <!-- DateTimePicker  -->
    <el-time-picker
      v-else-if="item.type === 'el-time-picker'"
      v-model="itemValue"
      :value-format="
        item.config['value-format'] ? item.config['value-format'] : 'timestamp'
      "
      :style="{ width: item.itemWidth ? item.itemWidth + 'px' : 'auto' }"
      v-bind="item.config"
      @change="onChange"
    >
    </el-time-picker>
  </div>
</template>

<script>
export default {
  name: "FilterItem",
  props: {
    // 单个表单元素配置项
    item: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      // 表单元素绑定值
      itemValue: null,
      // 父组件 filterData 表单元素数据
      itemData: null
    };
  },
  watch: {
    $attrs: {
      handler(val) {
        let itemData = val.value;
        // 引用类型拷贝
        if (this.isArrayOrObj(itemData)) itemData = this.jsonClone(itemData);
        this.itemValue = itemData;
      },
      immediate: true,
      deep: true
    }
  },
  methods: {
    onChange() {
      const itemValue = this.jsonClone(this.itemValue);
      this.$emit("update:value", itemValue); // 更新父组件 filterData
      this.$emit("change");
    },
    jsonClone(data) {
      return JSON.parse(JSON.stringify(data));
    },
    // 判断对象or数组
    isArrayOrObj(data) {
      let rst;
      if (Object.prototype.toString.call(data) === "[object Object]")
        rst = "object";
      if (Array.isArray(data)) rst = "array";

      return rst;
    }
  }
};
</script>