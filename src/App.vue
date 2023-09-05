<script setup lang="ts">
import zhCn from 'element-plus/es/locale/lang/zh-cn';

const locale = zhCn;
</script>
<template>
  <div>
    <el-config-provider :locale="locale">
      <router-view></router-view>
    </el-config-provider>
  </div>
</template>
