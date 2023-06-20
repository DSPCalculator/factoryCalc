<template>
  <div class="flex justify-start relative w-8">
    <ProductImg width="35" :imgKey="selectKey" class="inline-block" @click="changeFactory" />
    <div v-show="show" class="drop_down">
      <div v-for="(item, index) in props.selectList" :key="item.name">
        <ProductImg width="35" :imgKey="item.name" class="inline-block" @click="select(item.name, item.key)" />
      </div>
    </div>
  </div>
  <div v-show="show" class="overlay" @click="hidden"></div>
</template>

<script setup lang="ts">
import { PropType } from 'vue';
interface SelectListType {
  key: string;
  name: string;
  label: string;
}
interface Props {
  selectKey: string;
  selectType: string;
  selectList: SelectListType[];
  selectNum: string;
  selectFn: Function;
}
const props: Readonly<Props> = defineProps({
  selectKey: {
    type: String,
    required: true,
  },
  selectType: {
    type: String,
    required: true,
  },
  selectList: {
    type: Array as () => SelectListType[],
    required: true,
  },
  selectNum: {
    type: String,
    required: true,
  },
  selectFn: {
    type: Function as PropType<(key: string, type: string, sum: number) => void>,
    required: true,
  },
});
let list = props.selectList.filter((item) => item.key == props.selectKey);
if (props.selectList[props.selectNum]) {
  console.log(props.selectList, props.selectList.values);
  console.log(props.selectNum, props.selectNum);
}
let selectKey = '';
// TODO  代码毒瘤 这边取的逻辑实在是太让仍摸不着头脑了
// 希望有朝一日可以吧这段代码优化一下
//

if (props.selectType == 'additional_level') {
  // 增产剂等级和 建筑物两分逻辑

  let typeKet = {
    '0': 0,
    '1': 1,
    '2': 2,
    '3': 3,
    '4': 3,
  };
  selectKey = ref(props.selectList[typeKet[props.selectNum]].name);
} else {
  selectKey = ref(props.selectList[props.selectNum].name);
}
const show = ref(false);

const changeFactory = () => {
  show.value = true;
};
const hidden = () => {
  show.value = false;
};
const select = (key: string, index: string) => {
  show.value = false;
  selectKey.value = key;
  let typeKet = {
    '0': 0,
    '1': 1,
    '2': 2,
    '3': 4,
    '4': 4,
  };
  if (props.selectType == 'additional_level') {
    props.selectFn(props.selectKey, props.selectType, typeKet[index]);
  } else {
    console.log();

    props.selectFn(props.selectKey, props.selectType, index);
  }

  //  以上
};
</script>
<style lang="scss">
.drop_down {
  background-color: var(--bg-select-color);
  position: absolute;
  z-index: 1000;
  top: -3px;
  border-radius: 5%;
  padding: 8px 4px;
  text-align: center;
}
.drop_down div:hover {
  background-color: var(--bg-select-activate-color);
}
.overlay {
  position: fixed;
  top: 0;
  width: 100vw;
  height: 100vh;
  z-index: 999;
}
</style>
