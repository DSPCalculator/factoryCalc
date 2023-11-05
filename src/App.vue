<script setup lang="ts">
// import zhCn from 'element-plus/lib/locale/lang/zh-cn';

// const locale = zhCn;
</script>
<template>
  <div>
    <!-- :locale="locale" -->
    <el-config-provider>
      <router-view></router-view>
    </el-config-provider>
  </div>
</template>
