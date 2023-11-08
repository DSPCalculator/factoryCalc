<template>
  <div :class="theme.compact ? 'container-right-show' : 'container-right-hide'">
    <div class="container-right overflow-y-auto">
      <div class="add-items min-h-20">
        <div class="flex items-center justify-between h-10">
          <div class="flex items-center">
            <div v-if="!theme.compact" class="inline-block h-10 flex items-center justify-center">
              <el-tooltip class="box-item" :show-after="300" :auto-close="3000">
                <el-icon :size="18" class="mr-3 ml-5">
                  <button class="icon-btn mx-2 !outline-none" @click="click()">
                    <i-zondicons:indent-decrease v-if="theme.compact" class="icon-footer" />
                    <i-zondicons:indent-increase v-else class="icon-footer -rotate-x-180" />
                  </button>
                </el-icon>
              </el-tooltip>
            </div>

            <div class="inline-block ml-2">选择目标物</div>
            <ElButton class="ml-2" @click="openSelect('productList')">添加一个新产物</ElButton>
            <ElButton @click="openSelect('fixedProduction')">添加一条固定产线</ElButton>
            <ElButton
              v-show="Object.keys(productList).length > 0 || Object.keys(fixedProduction).length > 0"
              @click="cloneSelect"
            >
              清空产物列表</ElButton
            >
          </div>

          <div class="flex items-center justify-center">
            <dev>
              <el-select
                v-model="recipeName"
                ref="recipeRef"
                filterable
                @focus="tips"
                placeholder="添加保存配方"
                @change="recipeChange"
                @keyup.enter="addOption"
              >
                <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
                  <div class="flex justify-between">
                    <spen> {{ item.label }}</spen>
                    <el-button type="text" @click.stop="removeOption(item.value)">删除</el-button>
                  </div>
                </el-option>
              </el-select>
              <el-button type="primary" style="margin-left: 16px" @click="saveConfig">保存配置 </el-button>
            </dev>

            <el-tooltip :content="isDark ? t('change light') : t('change dark')" placement="top">
              <button class="flex items-center icon-btn mx-2 !outline-none" @click="toggleDark()">
                <i-ph-cloud-moon-bold v-if="isDark" class="icon-footer" />
                <i-ph-sun-horizon-bold v-else class="icon-footer" />
              </button>
            </el-tooltip>
            <el-button type="primary" style="margin-left: 16px" @click="changeConfigDrawer">常用配置项 </el-button>
          </div>
        </div>

        <div v-show="productList" class=" ">
          <!--  目标产线内容 -->
          <h2 class="pl-3">目标产线内容</h2>
          <div class="product flex items-center pl-5" v-for="key in Object.keys(productList)">
            <el-button type="primary" :icon="Delete" @click="delProduct('product', key)" />
            <ProductImg :key="key" :imgKey="key" class="mx-4" />
            <div class="mx-4 w-35 text-center">{{ DSP[key].name }}</div>
            <el-input-number v-model="productList[key]" controls-position="right" :min="0" class="mx-4 h-8" />
            / 分
            <!-- {{ config.productEfficiency }} -->

            <!-- / {{ config.productEfficiencyOption.filter((item) => item.value == config.productEfficiency) }} -->
          </div>
        </div>

        <div v-show="fixedProduction" class=" ">
          <!--  固定产线内容 -->
          <h2 class="pl-3">
            固定产线内容
            <el-tooltip content="固定产线可以以单个建筑的生产产量加入最终计算中" placement="top">
              <i-zondicons:exclamation-outline class="w-3"></i-zondicons:exclamation-outline>
            </el-tooltip>
          </h2>

          <div class="product flex items-center pl-5" v-for="target in fixedProduction">
            <el-button type="primary" :icon="Delete" @click="delProduct('fixed', target.targetName)" />
            <ProductImg :key="target.targetName" :imgKey="target.targetName" class="mx-4" />

            <div class="mx-4 w-35 text-center">{{ DSP[target.targetName].name }}</div>
            <div class="w-55">
              <el-input-number
                v-model="fixedProduction[target.targetName].targetSum"
                controls-position="right"
                :min="0"
                class="mx-4 h-8"
              />
              / 个
            </div>
            <!-- 工厂选择 -->
            <SelectFactory
              :selectKey="target.targetName"
              :selectList="
                facilityLabel(
                  recipeList.item_data[target.targetName],
                  recipeList.item_recipe_choices[target.targetName],
                )
              "
              :selectNum="target.architecture"
              selectType="architecture"
              :selectFn="changeFixedSpraying"
            >
            </SelectFactory>
            <!--  固定产线配方选择 -->
            <div v-for="(recipe, index) in recipeList.item_data[target.targetName]" :key="recipe">
              <!-- 配方的第0 个, 啥也不是, 冗余数据 ,需要跳出 -->
              <div
                v-if="index != 0 && recipeList.item_recipe_choices[target.targetName] == index"
                :class="{
                  active: recipeList.item_recipe_choices[target.targetName] == index,
                  'cursor-pointer': recipeList.item_data[target.targetName].length != 2,
                }"
                class="flex items-center m-2 w-85 pr-2 pl-2 rounded-lg bg-current"
                @click="changeRecipeOf(target.targetName, index, recipeList, 'fixed')"
              >
                <div v-for="material in Object.keys(recipeList.recipe_lists[recipe].in)">
                  <ProductImg
                    :imgKey="material"
                    class="inline-block mr-1"
                    :width="35"
                    :num="recipeList.recipe_lists[recipe].in[material]"
                  />
                </div>

                <div v-if="Object.keys(recipeList.recipe_lists[recipe].in).length > 0" class="text-cool-gray-50">
                  <i-zondicons:arrow-thin-right> </i-zondicons:arrow-thin-right>
                </div>
                <div v-for="product in Object.keys(recipeList.recipe_lists[recipe].out)">
                  <ProductImg
                    :imgKey="product"
                    class="inline-block mr-1"
                    :width="35"
                    :num="recipeList.recipe_lists[recipe].out[product]"
                  />
                </div>
                <span class="text-cool-gray-50" style="transform: scale(0.8)"
                  >({{ recipeList.recipe_lists[recipe].time }}s)
                  {{ recipeList.item_data[target.targetName].length == 2 ? '' : ' 点击选择配方' }}
                </span>
              </div>
            </div>
            <!-- 增产剂模式选择 -->
            <div class="production-model flex">
              <div
                v-for="option in config.miningIncOptions[
                  get_item_recipe_choices(target.targetName)['additional_mode_index']
                ]"
                class="ml-2 bg-current cursor-pointer miningInc rounded-lg"
                :class="{ active: target.additional_mode == option.key }"
                :key="option"
                @click="changeFixedSpraying(target.targetName, 'additional_mode', option.key)"
              >
                <span class="text-cool-gray-50"> {{ option.name }}</span>
              </div>
            </div>
            <!-- 增产剂等级-->

            <div v-if="target.additional_mode != 0">
              <SelectFactory
                :selectKey="target.targetName"
                :selectList="config.miningSprayingOptions"
                :selectNum="target.additional_level"
                selectType="additional_level"
                :selectFn="changeFixedSpraying"
              >
              </SelectFactory>
            </div>
          </div>
        </div>
        <div v-if="Object.keys(recipeList.mineralize_list).length > 0">
          原矿化的产物: (单击图标删除原矿化物品)
          <div class="flex">
            <div v-for="mineralize in Object.keys(recipeList.mineralize_list)" class="">
              <ProductImg :key="key" :imgKey="mineralize" class="mx-4" @click="del_mine(mineralize)" />
            </div>
          </div>
        </div>
      </div>
      <DSPElDialog
        :title="dialogTitle"
        :visible="dialogFormVisible"
        :close="closeProduct"
        :clickSelect="selectProduct"
      ></DSPElDialog>
      <!-- 配方详情列表内容 -->
      <div>
        <div class="lists min-h-35">
          <div class="sticky top-0 z-99 list-title">
            <ul class="flex text-center">
              <li class="btn-action">操作</li>
              <li class="target-product">目标产物</li>
              <li class="demand">需求量</li>
              <li class="plant">所需工厂</li>
              <li class="recipe">配方选择</li>
              <li class="production">工厂类型选择</li>
              <li class="production-model">增产模式选择</li>
            </ul>
          </div>
          <div
            v-show="recipeList.list_data.length === 0"
            class="flex justify-center items-center"
            style="height: 115px"
          >
            展示产物信息
          </div>

          <!-- 循环列表开始 -->
          <div v-for="result in recipeList.list_data" :key="result" class="list-content flex text-center border-dashed">
            <div class="btn-action btn flex justify-center items-center">
              <el-Button plain size="small" @click="set_mine(result.key)"
                >{{ recipeList.mineralize_list[result.key] ? '取消原矿' : '视为原矿' }}
              </el-Button>
            </div>
            <div class="target-product flex justify-center items-center">
              <ProductImg :imgKey="result.key" class="inline-block" />
            </div>
            <div class="demand">
              {{ result.efficiency }}
              <div v-if="Object.keys(recipeList.excessProduct).length > 0 && recipeList.excessProduct[result.key]">
                <div v-for="excessProduct in Object.keys(recipeList.excessProduct[result.key])">
                  +来自 {{ DSP[excessProduct].name }} 的 {{ recipeList.excessProduct[result.key][excessProduct] }}
                </div>
              </div>
            </div>
            <!-- 所需要的工厂 -->
            <div class="plant flex items-center">
              <ProductImg :imgKey="result.factoriesNum.key" class="inline-block" />
              {{ result.factoriesNum.num }}
            </div>
            <!-- 选择的配方 -->
            <div class="recipe flex-col" v-if="!result.factoriesNum.is_mineralized">
              <div v-for="(recipe, index) in recipeList.item_data[result.key]" :key="recipe">
                <!-- 配方的第0 个, 啥也不是, 冗余数据 ,需要跳出 -->
                <div
                  v-if="index != 0 && recipeList.item_recipe_choices[result.key] == index"
                  :class="{
                    active: recipeList.item_recipe_choices[result.key] == index,
                    'cursor-pointer': recipeList.item_data[result.key].length != 2,
                  }"
                  class="flex items-center m-2 w-85 pr-2 pl-2 rounded-lg bg-current"
                  @click="changeRecipeOf(result.key, index, recipeList, 'goal')"
                >
                  <div v-for="material in Object.keys(recipeList.recipe_lists[recipe].in)">
                    <ProductImg
                      :imgKey="material"
                      class="inline-block mr-1"
                      :width="35"
                      :num="recipeList.recipe_lists[recipe].in[material]"
                    />
                  </div>

                  <div v-if="Object.keys(recipeList.recipe_lists[recipe].in).length > 0" class="text-cool-gray-50">
                    <i-zondicons:arrow-thin-right> </i-zondicons:arrow-thin-right>
                  </div>
                  <div v-for="product in Object.keys(recipeList.recipe_lists[recipe].out)">
                    <ProductImg
                      :imgKey="product"
                      class="inline-block mr-1"
                      :width="35"
                      :num="recipeList.recipe_lists[recipe].out[product]"
                    />
                  </div>
                  <span class="text-cool-gray-50" style="transform: scale(0.8)"
                    >({{ recipeList.recipe_lists[recipe].time }}s)
                    {{ recipeList.item_data[result.key].length == 2 ? '' : ' 点击选择配方' }}
                  </span>
                </div>
              </div>
            </div>
            <!-- 工厂选择 -->
            <div class="flex plant-model justify-start" v-if="!result.factoriesNum.is_mineralized">
              <div
                class="bg-current ml-4 p-1 rounded-lg cursor-pointer"
                :class="{ active: get_item_recipe_choices(result.key)['architecture'] == index }"
                v-for="(facility, index) in facilityLabel(
                  recipeList.item_data[result.key],
                  recipeList.item_recipe_choices[result.key],
                )"
                @click="changeRecipeRecipeChoices(result.key, 'architecture', index)"
              >
                <ProductImg width="35" :imgKey="facility.name" class="inline-block" />
              </div>
            </div>
            <!-- 选择增产剂模式 -->
            <!-- && get_item_recipe_choices(result.key)['additional_level'] != 0 -->
            <div class="production-model flex" v-if="!result.factoriesNum.is_mineralized">
              <div
                v-for="option in config.miningIncOptions[get_item_recipe_choices(result.key)['additional_mode_index']]"
                class="ml-2 bg-current cursor-pointer miningInc rounded-lg"
                :class="{ active: get_item_recipe_choices(result.key)['additional_mode'] == option.key }"
                :key="option"
                @click="changeRecipeRecipeChoices(result.key, 'additional_mode', option.key)"
              >
                <span class="text-cool-gray-50"> {{ option.name }}</span>
              </div>
            </div>
            <!-- 增产剂等级 -->
            <div
              class="production ml-5 flex"
              v-if="!result.factoriesNum.is_mineralized && get_item_recipe_choices(result.key)['additional_mode'] != 0"
            >
              <SelectFactory
                :selectKey="result.key"
                :selectList="config.miningSprayingOptions"
                :selectNum="get_item_recipe_choices(result.key)['additional_level']"
                selectType="additional_level"
                :selectFn="changeRecipeRecipeChoices"
              >
              </SelectFactory>
            </div>

            <div v-if="0 != Number(result.factoriesNum)"></div>
          </div>
        </div>
      </div>
      <div class="pb-5 pl-20">
        <template v-if="Object.keys(recipeList.lp_surplus_list).length > 0">
          <h2>冗余产物</h2>
          <div v-for="building in Object.keys(recipeList.lp_surplus_list)" class="flex items-center">
            <ProductImg width="35" :imgKey="building" class="inline-block" />
            {{ DSP[building].name }} --- {{ recipeList.lp_surplus_list[building] }} 个
          </div>
        </template>
      </div>
      <div class="h-[calc(100%-50rem)] pb-5 pl-20">
        <template v-if="Object.keys(recipeList.building_list).length > 0">
          <div>总计需要的建筑:</div>
          <div v-for="building in Object.keys(recipeList.building_list)" class="flex items-center">
            <ProductImg width="35" :imgKey="building" class="inline-block" />
            {{ DSP[building].name }} --- {{ recipeList.building_list[building] }} 个
          </div>
          <div>
            预估电力需求下限：{{ recipeList.energy_cost.toFixed(2) }} MW
            <el-button color="#626aef" :dark="isDark" @click="change_energy">
              {{ config.energy_contain_miner ? '忽视采集设备耗电' : '考虑采集设备耗电' }}</el-button
            >
          </div>
        </template>

        <EnReadme class=""></EnReadme>
        <span>
          {{ devModel ? productList : '' }}
        </span>
      </div>
    </div>
    <FormulaSelectionDialog
      :visible="visibleFormulaDialog"
      :close="closeformuldialog"
      :date="FormulaDialogDate"
      :dateKey="FormulaDialogDateKey"
      :changeRecipeOf="tagRecipeType === 'goal' ? change_recipe_of : fixed_change_recipe_of"
    ></FormulaSelectionDialog>
    <ConfigDrawer :show="configDrawerShow" :close="changeConfigDrawer"></ConfigDrawer>
  </div>
</template>

<script setup lang="ts">
import { isDark, toggleDark } from '@/utils/dark';
import { reactive, markNonReactive } from 'vue';
import { Delete } from '@element-plus/icons-vue';
import useCounterStore from '@/store/theme';
import useConfigStore from '@/store/config';
import DSP from '@/assets/data/DSP.json';
import game_data from '@/assets/data/game_data.json';
import {
  calculate,
  get_item_recipe_choices,
  set_item_recipe_choices,
  change_recipe_of,
  mineralize,
  unMineralize,
  get_one_item_recipe_choices,
  update_fixed_recipe,
} from '@/utils/calculate';
const theme = useCounterStore();
const config = useConfigStore();
let dialogFormVisible = ref(false); //显示dialog 框
let devModel = ref(import.meta.env.DEV); //显示dialog 框

const visibleFormulaDialog = ref(false);
const FormulaDialogDate = ref([]);
const FormulaDialogDateKey = ref(0);
const click = () => {
  // 点击切换侧边栏状态;
  theme.changeCompact();
};
const recipeRef = ref(null);
const tips = () => {
  ElMessage({
    message: '输入后按下回车即可保存',
    type: 'success',
  });
};
const recipeName = ref('');
const options = ref(config.recipeList);

const addOption = (event) => {
  const input = event.target.value;
  console.log('recipeRef', toRaw(recipeRef));
  if (input && !options.value.find((option) => option.value === input)) {
    options.value.push({ value: input, label: input });
    recipeName.value = input;
  }
};
const saveConfig = () => {
  console.log('recipeName.value', recipeName.value);

  if (recipeName.value.length > 0) {
    let rawProductList = toRaw(productList.value);
    let rawFixedProduction = toRaw(fixedProduction.value);
    let key = toRaw(recipeName.value);
    let copyConfig = JSON.parse(JSON.stringify(config));
    config.recipeListDate[key] = {
      productList: rawProductList,
      fixedProduction: rawFixedProduction,
      config: {
        scienceResearchSpeed: copyConfig.scienceResearchSpeed,
        miniCore: copyConfig.miniCore,
        largeCore: copyConfig.largeCore,
        largeCoreWorkingSpeed: copyConfig.largeCoreWorkingSpeed,
        oilWellSpeed: copyConfig.oilWellSpeed,
        hydrogenCollectionRate: copyConfig.hydrogenCollectionRate,
        heavyHydrogenCollectionRate: copyConfig.heavyHydrogenCollectionRate,
        combustibleIceCollectionRate: copyConfig.combustibleIceCollectionRate,
        shooter: copyConfig.shooter,
        fractionatingColumnSpeed: copyConfig.fractionatingColumnSpeed,
        energy_contain_miner: copyConfig.energy_contain_miner,
        defaultMining: copyConfig.defaultMining,
        defaultSmelting: copyConfig.defaultSmelting,
        defaultProduction: copyConfig.defaultProduction,
        defaultChemical: copyConfig.defaultChemical,
        defaultCharge: copyConfig.defaultCharge,
        defaultSpraying: copyConfig.defaultSpraying,
        defaultInc: copyConfig.defaultInc,
      },
    };
  }
};

const removeOption = (removeValue) => {
  options.value = options.value.filter((option) => option.value !== removeValue);
  config.recipeList = config.recipeList.filter((option) => option.value !== removeValue);
  if (recipeName.value === removeValue) {
    recipeName.value = '';
  }
  delete config.recipeListDate[removeValue];
};
const configDrawerShow = ref(false);
const changeConfigDrawer = () => {
  configDrawerShow.value = !configDrawerShow.value;
};
const recipeChange = (recipeKey) => {
  console.log('recipeKey', recipeKey);

  //  保存的所有配方进行切换
  if (recipeKey == '') {
    console.log('没有获取到参数, 故此跳出');

    // 置空 选择配方
    return;
  }
  if (Object.keys(config.recipeListDate).length > 0 && recipeKey in config.recipeListDate) {
    let data = JSON.parse(JSON.stringify(config.recipeListDate[recipeKey]));
    console.log('toRaw data', data);

    if (data['productList']) {
      productList.value = toRaw(data['productList']);
    }
    if (data['fixedProduction']) {
      fixedProduction.value = toRaw(data['fixedProduction']);
    }

    config.setConfig(data['config']);
  }
};
const { t, availableLocales, locale } = useI18n();
console.log();
const set_mine = (key) => {
  if (recipeList.value.mineralize_list[key]) {
    unMineralize(key);
  } else {
    mineralize(key);
  }
  config.changeConfig();
};
const del_mine = (key) => {
  unMineralize(key);
  config.changeConfig();
};
const fixed_change_recipe_of = (key, index) => {
  console.log('fixedProduction', key, index);
  fixedProduction.value[key]['recipe_id'] = index;
  change_recipe_of(key, index);
};
const openSelect = (key: string) => {
  console.log('dialogFormVisible', dialogFormVisible.value);
  dialogFormVisible.value = !dialogFormVisible.value;
  if (key == 'productList') {
    dialogTitle.value = '选择目标产物';
  } else {
    dialogTitle.value = '选择固定产线';
  }
};
const changeUnitTime = (sum) => {
  let Num = sum;
  console.log('默认单位配置,', config.productEfficiency);

  switch (config.productEfficiency) {
    case 's':
      Num = sum * 60;
      break;
    case 'h':
      Num = sum / 60;
      break;
    default:
      break;
  }
  return Num;
};
const selectProduct = (selectItem: { key: string; num: number }) => {
  console.log('selectItem', selectItem);
  recipeName.value = '';

  console.log('===================置空', recipeName.value);
  // 将保存配方的名字置空
  // recipeName.value = '';

  // selectItem['num'] = changeUnitTime(selectItem['num']);

  if (dialogTitle.value == '选择目标产物') {
    if (selectItem.key in productList.value) {
      productList.value[selectItem.key] += selectItem.num;
    } else {
      productList.value[selectItem.key] = selectItem.num;
    }
  } else {
    // 固定产物添加
    if (selectItem.key in fixedProduction.value) {
      fixedProduction.value[selectItem.key].targetSum += selectItem.num;
    } else {
      fixedProduction.value[selectItem.key] = {
        targetName: selectItem.key,
        targetSum: selectItem.num,
        recipe_id: 1,
        additional_level: 4,
        additional_mode: 0,
        architecture: 0,
      };
    }
    update_fixed_recipe(fixedProduction.value);
    config.changeConfig();
  }

  dialogFormVisible.value = false;
};

const closeProduct = () => {
  // console.error('❌%c1111', 'color: red; font-size: 20px');
  calculate();
  dialogFormVisible.value = false;
};
// 当前选中的目标产物列表
const productList = ref({});
//固定产线选择
const fixedProduction = ref({});

// 修改生产模式模式
const changeFixedSpraying = (key, type, index) => {
  if (type === 'additional_mode') {
    if (fixedProduction.value[key][type] === index) {
      fixedProduction.value[key][type] = 0;
    } else {
      fixedProduction.value[key][type] = index;
    }
  } else {
    if (fixedProduction.value[key]) {
      fixedProduction.value[key][type] = index;
    }
  }
};
// 修改生产模式模式
const fexchangeFixedSpraying = (key, type, index) => {
  console.log(key, type, index);
  get_item_recipe_choices(key, type, index);
  config.changeConfig();
};
const dialogTitle = ref('选择目标产物');
const tagRecipeType = ref('goal');
const obj = ref(false);
//点击删除产物
const delProduct = (type: string, key: string) => {
  recipeName.value = '';

  if (type == 'fixed') {
    // 固定产物
    if (fixedProduction.value[key]) {
      // del_fixed_recipe(key);
      delete fixedProduction.value[key];
      config.changeConfig();
    }
  } else {
    if (productList.value[key]) {
      delete productList.value[key];
    }
  }
};
const cloneSelect = () => {
  recipeName.value = '';
  productList.value = {};
  fixedProduction.value = {};
};
watch(fixedProduction, async (newQuestion, oldQuestion) => {
  config.changeConfig();
});
//工厂修改器
const changeFactory = () => {
  console.log('1111');
};

// 在list 找到相应的数据
const facilityLabel = (list, key) => {
  return game_data['factory_data'][recipeList.value.recipe_lists[list[key]]['facility']];
};
const recipeList = computed(() => {
  let newData = {};
  console.log('计算的需求信息', productList.value);
  // for (const key in productList.value) {
  //   if (Object.prototype.hasOwnProperty.call(productList.value, key)) {
  //     const element = productList.value[key];
  //     console.log(element);

  //     // newData[key] = changeUnitTime(element);
  //   }
  // }
  // console.log('newData', newData);

  const data = calculate(productList.value);
  // TODO 这个 console  绑定了依赖项， 如果删除，则丢失，不能相应批量改变
  console.log('数据发生改变，重新计算所有依赖', config.isChange);
  console.log('data', data);
  // data.list_data.sost((a) => a.factoriesNum.is_mineralized);

  return data;
});

const changeRecipeOf = (key, index, array, type) => {
  if (array.item_data[key].length != 2) {
    visibleFormulaDialog.value = true;
    FormulaDialogDate.value = array;
    FormulaDialogDateKey.value = key;
    tagRecipeType.value = type;
  } else {
    ElMessage({
      message: '此产物没有配方选择',
      type: 'warning',
    });
  }
  config.changeConfig();
};
const closeformuldialog = () => {
  visibleFormulaDialog.value = false;
  // FormulaDialogDate.value = [];
  // FormulaDialogDateKey.value = 0;
};
const changeRecipeRecipeChoices = (key, type, index) => {
  // 修改增产剂等级
  if (get_one_item_recipe_choices(key, type) === index) {
    set_item_recipe_choices(key, type, 0);
  } else {
    set_item_recipe_choices(key, type, index);
  }

  config.changeConfig();
};
const restorConfig = () => {
  config.restoreMinings();
  config.restoreFactory();
  productList.value = {};
};

const change_energy = () => {
  config.change_energy_contain_miner();
  config.changeConfig();
};
</script>
<style lang="scss">
.container-right-show {
  padding-left: 420px;
}
.container-right-show {
  margin-left: 0px;
}
.miningInc {
  font-size: 12px;
  padding: 4px 8px;
}
// 未选择的背景色
.bg-current {
  background-color: #878787;
}
.top {
  background-color: red;
}

.container-right {
  border-bottom: #8c949c solid 2px;
}

.lists {
  border-bottom: #8c949c solid 2px;
}
.lists > div {
}
.list-title {
  position: sticky;
  top: 0;
  border-bottom: #42586c solid 2px;
}
.list-content {
  border-bottom: dashed 2px #7386aa;
}

.list-content:nth-last-child(1) {
  border: none;
}
.list-content {
  display: flex;
  align-items: center;
}

.icon-footer {
  font-size: 1.3em;
}

.product {
  border-bottom: 2px solid #222;
}

.product:nth-last-child(1) {
  // border: none;
}
.active {
  background-color: #42586c;
}

.btn-action {
  width: 6%;
}
.target-product {
  width: 6%;
}
.demand {
  width: 10%;
  display: flex;
  flex-direction: column;
}
.plant {
  width: 7%;
}
.list-content .plant {
  /* justify-content: space-evenly; */
}
/* 配方的宽度百分比 */
.recipe {
  min-width: 356px;
}
.plant-model {
  min-width: 190px;
}
.production {
  min-width: 200px;
}
.production-model {
  min-width: 120px;
}
</style>
