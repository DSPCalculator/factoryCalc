<template>
  <el-drawer v-model="props.show" title="计算器页面配置项" :with-header="true" @close="props.close">
    <!-- <div><span class="mr-3">大字模式开启</span> <el-switch v-model="largeFont" size="large" /></div>
    <div><span class="mr-3">弹出页模式</span> <el-switch v-model="largeFont2" size="large" /></div>
    <div><span class="mr-3">进阶模式</span> <el-switch v-model="largeFont3" size="large" /></div>
    <div><span class="mr-3">默认添加增产剂会选择三级增产剂</span> <el-switch v-model="largeFont4" size="large" /></div>
    <div><span class="mr-3">默认修改增产模式为</span> 增产 <el-switch v-model="largeFont5" size="large" /></div>
    <div><span class="mr-3">修改默认添加产物数量</span><el-switch v-model="largeFont6" size="large" /></div> -->
    <div class="mr-3">
      修改产物计算速度单位为
      <el-select v-model="value" class="m-2" placeholder="Select" size="large" @change="config.changeProductEfficiency">
        <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
      </el-select>
    </div>
  </el-drawer>
</template>

<script lang="ts" setup>
import useConfigStore from '@/store/config';
const config = useConfigStore();
const props = defineProps({
  show: Boolean,
  close: Function,
});

const largeFont = ref(false);
const largeFont2 = ref(false);
const largeFont3 = ref(false);
const largeFont4 = ref(false);
const largeFont5 = ref(false);
const largeFont6 = ref(false);
// console.log('111');

const value = ref(config.productEfficiency);
const options = reactive(config.productEfficiencyOption);
const productChange = (value) => {
  console.log('value', value);
};
</script>
