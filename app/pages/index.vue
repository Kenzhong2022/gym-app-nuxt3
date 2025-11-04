<template>
  <div
    class="min-h-screen bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center p-4"
  >
    <!-- 侧边菜单按钮 -->
    <div class="fixed top-4 left-4 z-50">
      <button
        @click="toggleMenu"
        class="bg-white rounded-lg shadow-lg p-3 hover:shadow-xl transition-all duration-200 hover:scale-105"
      >
        <div
          class="w-6 h-6 flex flex-col justify-center items-center space-y-1"
        >
          <div class="w-full h-0.5 bg-gray-800 rounded-full"></div>
          <div class="w-full h-0.5 bg-gray-800 rounded-full"></div>
          <div class="w-full h-0.5 bg-gray-800 rounded-full"></div>
        </div>
      </button>

      <!-- 下拉菜单 -->
      <div
        v-if="isMenuOpen"
        class="absolute top-16 left-0 bg-white rounded-lg shadow-xl py-2 w-48 transition-all duration-300 transform origin-top-left"
        :class="isMenuOpen ? 'scale-100 opacity-100' : 'scale-95 opacity-0'"
      >
        <button
          @click="handleMenuItem('home')"
          class="w-full px-4 py-3 text-left hover:bg-gray-100 transition-colors duration-200 text-gray-800 font-medium"
        >
          🏠 首页
        </button>
        <button
          @click="handleMenuItem('lottery')"
          class="w-full px-4 py-3 text-left hover:bg-gray-100 transition-colors duration-200 text-gray-800 font-medium"
        >
          🎲 抽奖
        </button>
        <button
          @click="handleMenuItem('settings')"
          class="w-full px-4 py-3 text-left hover:bg-gray-100 transition-colors duration-200 text-gray-800 font-medium"
        >
          ⚙️ 设置
        </button>
        <div class="border-t border-gray-200 my-1"></div>
        <button
          @click="handleMenuItem('logout')"
          class="w-full px-4 py-3 text-left hover:bg-red-50 hover:text-red-600 transition-colors duration-200 text-gray-800 font-medium"
        >
          🚪 退出
        </button>
      </div>
    </div>

    <!-- 设置对话框 -->
    <el-dialog
      v-model="dialogVisible"
      title="设置"
      width="600px"
      :before-close="handleClose"
    >
      <h3 class="text-lg font-medium mb-4">奖品管理</h3>
      <!-- 位置变化提示 -->
      <div
        class="mb-4 p-3 bg-blue-50 border border-blue-200 rounded-lg"
        v-if="isEditMode"
      >
        <p class="text-sm text-blue-800">
          <span class="font-medium">💡 提示：</span>
          更改奖品位置时，如果目标位置已被占用，系统会自动将冲突的奖品移动到第一个空闲可用位置。[顺时针从左上角开始是
          0 ]
          {{ tempPrizeList[0] }}
          {{ tempPrizeList[1] }}
        </p>
      </div>
      <!-- 奖品设置 -->
      <div class="p-4">
        <!-- 奖品表格 -->
        <el-table
          :data="isEditMode ? tempPrizeList : prizeList"
          style="width: 100%"
          class="mb-4 prize-table"
          row-class-name="prize-row"
        >
          <el-table-column prop="id" label="ID" width="60" />
          <el-table-column label="奖品名称" width="180">
            <template #default="scope">
              <el-input
                v-if="isEditMode"
                v-model="scope.row.name"
                size="small"
                placeholder="请输入奖品名称"
              />
              <span v-else>{{ scope.row.name }}</span>
            </template>
          </el-table-column>
          <el-table-column label="必中" width="80">
            <template #default="scope">
              <el-switch
                v-model="scope.row.required"
                @change="handleRequiredChange(scope.row)"
                :disabled="!isEditMode"
              />
            </template>
          </el-table-column>
          <el-table-column label="位置" width="80">
            <template #default="scope">
              <el-select
                v-if="isEditMode"
                v-model="scope.row.index"
                size="small"
                placeholder="位置"
                @change="handlePositionChange(scope.row, scope.$index)"
              >
                <el-option
                  v-for="pos in availablePositions"
                  :key="pos"
                  :label="pos"
                  :value="pos"
                />
              </el-select>
              <span v-else>{{ scope.row.index }}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="100">
            <template #default="scope">
              <el-button
                size="small"
                type="danger"
                @click="deletePrize(scope.$index)"
                :disabled="!isEditMode"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <!-- 编辑模式切换按钮 -->
        <div class="mb-4 flex justify-end">
          <el-button
            v-if="isEditMode"
            @click="cancelEditMode"
            size="small"
            class="mr-2"
          >
            取消编辑
          </el-button>
          <el-button
            :type="isEditMode ? 'success' : 'primary'"
            @click="toggleEditMode"
            size="small"
          >
            {{ isEditMode ? "保存编辑" : "编辑模式" }}
          </el-button>
        </div>
        <!-- 新增奖品表单 -->
        <div class="flex gap-2 mb-4" v-if="isEditMode">
          <el-input
            v-model="newPrize.name"
            placeholder="奖品名称"
            style="flex: 1"
          />
          <el-button type="primary" @click="addPrize">新增</el-button>
        </div>
      </div>
      <h3 class="text-lg font-medium mb-4">奖品预览</h3>
      <!-- 九宫格奖品位置预览 -->
      <div class="grid grid-cols-3 gap-2">
        <div
          v-for="pos in 9"
          :key="pos"
          class="bg-gray-200 flex items-center justify-center p-2 text-sm rounded-md"
          :class="{ 'bg-gray-400': pos === 5 }"
        >
          <template v-if="pos === 5"> 中心按钮 </template>
          <template v-else>
            {{ getPrizeNameByPosition(pos) }}
          </template>
        </div>
      </div>

      <template #footer>
        <span class="dialog-footer">
          <el-button size="large" @click="handleCancel">返回首页</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 抽奖组件 -->
    <kk-lettery
      ref="lotteryRef"
      :list="prizeList"
      :isRandom="false"
      @lottery-start="handleLotteryStart"
      @lottery-end="handleLotteryEnd"
    />
  </div>
</template>

<script setup>
import { ref, watch, computed, nextTick } from "vue";

// 菜单状态
const isMenuOpen = ref(false);

// 对话框状态
const dialogVisible = ref(false);

// 设置状态
const settings = ref({
  sound: true,
  speed: "normal",
});

/**
 * "九宫格" 8 个奖品格按从左到右、从上到下跳过中心按钮的顺序，依次对应后端 prizes 数组的第 0~7 号元素：
 * 1 号格 → prizes[0]
 * 2 号格 → prizes[1]
 * 3 号格 → prizes[2]
 * 4 号格 → prizes[7]
 * 6 号格 → prizes[3]
 * 7 号格 → prizes[6]
 * 8 号格 → prizes[5]
 * 9 号格 → prizes[4]
 */
// 奖品列表
const prizeList = ref([
  { id: 1, name: "一等奖 🏆", required: false, index: 0 },
  { id: 2, name: "二等奖 🥈", required: false, index: 1 },
  { id: 3, name: "三等奖 🥉", required: false, index: 2 },
  { id: 4, name: "四等奖 🎁", required: false, index: 3 },
  { id: 5, name: "五等奖 🎈", required: false, index: 5 },
  { id: 6, name: "六等奖 🎊", required: false, index: 6 },
  { id: 7, name: "七等奖 🎉", required: false, index: 7 },
  { id: 8, name: "谢谢参与 💝", required: true, index: 8 },
]);

// 监听奖品列表变化
watch(
  prizeList,
  (newList, oldList) => {
    console.log("[奖品管理] 奖品列表发生变化:");
    console.log("[奖品管理] 新列表:", JSON.parse(JSON.stringify(newList)));
    console.log("[奖品管理] 旧列表:", JSON.parse(JSON.stringify(oldList)));

    // 统计必中奖品
    const requiredPrizes = newList.filter((prize) => prize.required);
    console.log(`[奖品管理] 当前必中奖品数量: ${requiredPrizes.length}`);
    if (requiredPrizes.length > 0) {
      console.log(
        "[奖品管理] 必中奖品:",
        requiredPrizes.map((p) => p.name).join(", ")
      );
    }
  },
  { deep: true }
);

// 新增奖品表单
const newPrize = ref({
  name: "",
});

// 抽奖组件引用
const lotteryRef = ref(null);

// 菜单切换
const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

// 对话框关闭处理
const handleClose = (done) => {
  console.log("[设置] 关闭对话框");
  done();
};

// 菜单项点击处理
const handleMenuItem = (item) => {
  console.log(`[菜单] 点击了: ${item}`);
  isMenuOpen.value = false;

  switch (item) {
    case "home":
      console.log("[菜单] 返回首页");
      break;
    case "lottery":
      console.log("[菜单] 重新开始抽奖");
      if (lotteryRef.value) {
        lotteryRef.value.resetLottery();
      }
      break;
    case "settings":
      console.log("[菜单] 打开设置");
      dialogVisible.value = true;
      break;
    case "logout":
      console.log("[菜单] 退出登录");
      break;
  }
};

// 抽奖开始事件处理
const handleLotteryStart = () => {
  console.log("[抽奖] 抽奖开始");
};

// 抽奖结束事件处理
const handleLotteryEnd = (result) => {
  console.log("[抽奖] 抽奖结束，结果:", result);
};

// 点击外部关闭菜单
const closeMenu = () => {
  isMenuOpen.value = false;
};

// 编辑模式状态
const isEditMode = ref(false);

// 临时奖品列表（用于编辑）
const tempPrizeList = ref([]);

// 可用位置列表（0-8，跳过中心位置4）
const availablePositions = computed(() => {
  return [0, 1, 2, 3, 5, 6, 7, 8];
});

// 根据九宫格位置获取对应的奖品名称
const getPrizeNameByPosition = (position) => {
  // 位置映射关系（根据注释中的映射）
  const positionMap = {
    1: 0, // 1号格 → prizes[0]
    2: 1, // 2号格 → prizes[1]
    3: 2, // 3号格 → prizes[2]
    4: 7, // 4号格 → prizes[7]
    6: 3, // 6号格 → prizes[3]
    7: 6, // 7号格 → prizes[6]
    8: 5, // 8号格 → prizes[5]
    9: 4, // 9号格 → prizes[4]
  };

  const prizeIndex = positionMap[position];
  // 使用临时奖品列表（编辑模式下）或原始奖品列表
  const currentList = isEditMode.value ? tempPrizeList.value : prizeList.value;
  if (prizeIndex !== undefined && currentList[prizeIndex]) {
    return currentList[prizeIndex].name;
  }
  return "空";
};

// 切换编辑模式
const toggleEditMode = () => {
  if (isEditMode.value) {
    // 保存编辑 - 将临时数据复制到真实数据
    // 检查临时数组长度是否为8
    if (tempPrizeList.value.length !== 8) {
      ElMessage.warning("奖品数量必须为8个才能保存并退出编辑");
      return;
    }
    prizeList.value = JSON.parse(JSON.stringify(tempPrizeList.value));
    console.log("[编辑模式] 保存编辑，更新真实奖品列表");
    isEditMode.value = false;
    ElMessage.success("编辑保存成功");
  } else {
    // 进入编辑模式 - 复制真实数据到临时数据
    tempPrizeList.value = JSON.parse(JSON.stringify(prizeList.value));
    console.log("[编辑模式] 进入编辑模式，复制数据到临时变量");
    isEditMode.value = true;
  }
};

// 处理必中状态变化（编辑模式下）
const handleRequiredChange = (row) => {
  if (!isEditMode.value) return;

  console.log(`[编辑模式] 奖品 "${row.name}" 必中状态: ${row.required}`);
  if (row.required) {
    // 确保只有一个必中项（在临时数据中）
    tempPrizeList.value.forEach((prize) => {
      if (prize.id !== row.id) {
        prize.required = false;
      }
    });
  }
};

// 新增奖品（编辑模式下）
const addPrize = () => {
  if (!isEditMode.value) return;

  // 检查是否已达到最大奖品数量（8个）
  if (tempPrizeList.value.length >= 8) {
    ElMessage.warning("奖品数量已达到最大值（8个），无法继续添加");
    return;
  }

  if (!newPrize.value.name) {
    ElMessage.warning("请输入奖品名称");
    return;
  }

  const newId =
    tempPrizeList.value.length > 0
      ? Math.max(...tempPrizeList.value.map((p) => p.id)) + 1
      : 1;

  // 找到第一个可用的位置
  const usedPositions = tempPrizeList.value.map((p) => p.index);
  const availablePos = availablePositions.value.find(
    (pos) => !usedPositions.includes(pos)
  );

  if (availablePos === undefined) {
    ElMessage.warning("没有可用的位置了");
    return;
  }

  tempPrizeList.value.push({
    id: newId,
    name: newPrize.value.name,
    required: false,
    index: availablePos,
  });

  // 重置表单
  newPrize.value = {
    name: "",
  };

  console.log(
    "[编辑模式] 新增奖品:",
    tempPrizeList.value[tempPrizeList.value.length - 1]
  );
};

// 删除奖品（编辑模式下）
const deletePrize = (index) => {
  if (!isEditMode.value) return;

  const deletedPrize = tempPrizeList.value[index];
  tempPrizeList.value.splice(index, 1);
  console.log("[编辑模式] 删除奖品:", deletedPrize);
};

// 取消编辑模式
const cancelEditMode = () => {
  // 恢复临时数据为原始数据
  tempPrizeList.value = JSON.parse(JSON.stringify(prizeList.value));
  isEditMode.value = false;
  ElMessage.info("已取消编辑，数据已恢复");
};

//
const handleCancel = () => {
  if (isEditMode.value) {
    // 退出编辑模式，不保存更改
    isEditMode.value = false;
    tempPrizeList.value = [];
    console.log("[编辑模式] 取消编辑，不保存更改");
  }
  dialogVisible.value = false;
};

// 保存设置（现在只有在编辑模式下才能保存）
const saveSettings = () => {
  if (!isEditMode.value) {
    console.log("[编辑模式] 非编辑模式，无法保存");
    return;
  }

  // 检查奖品数量是否为8个
  if (tempPrizeList.value.length !== 8) {
    ElMessage.warning("奖品数量必须为8个才能保存");
    return;
  }

  // 保存编辑 - 将临时数据复制到真实数据
  prizeList.value = JSON.parse(JSON.stringify(tempPrizeList.value));
  console.log("[编辑模式] 保存设置，当前奖品列表:", prizeList.value);
  dialogVisible.value = false;
  isEditMode.value = false;
  ElMessage.success("设置保存成功");
};

// 交换数组元素并触发动画
const swapArrayElements = async (index1, index2) => {
  if (index1 === index2) return;

  // 添加交换动画类
  const table = document.querySelector(".prize-table");
  if (table) {
    table.classList.add("swapping");
  }

  // 交换数组元素的所有内容（包括位置信息）
  console.log("[数组交换] 开始交换前的数组状态:");
  console.log(
    "[数组交换] index1:",
    index1,
    "数据:",
    tempPrizeList.value[index1]
  );
  console.log(
    "[数组交换] index2:",
    index2,
    "数据:",
    tempPrizeList.value[index2]
  );

  // 保存两个元素的完整数据
  const temp = { ...tempPrizeList.value[index1] };

  // 交换数组位置
  tempPrizeList.value[index1] = { ...tempPrizeList.value[index2] };
  tempPrizeList.value[index2] = temp;

  console.log("[数组交换] 交换完成后的数组状态:");
  console.log(
    "[数组交换] index1:",
    index1,
    "数据:",
    tempPrizeList.value[index1]
  );
  console.log(
    "[数组交换] index2:",
    index2,
    "数据:",
    tempPrizeList.value[index2]
  );

  // 等待DOM更新
  await nextTick();

  // 移除动画类
  setTimeout(() => {
    if (table) {
      table.classList.remove("swapping");
    }
  }, 300);
};

// 处理位置变化（编辑模式下）- 实现真正的数组下标交换
const handlePositionChange = async (changedPrize, changedIndex) => {
  if (!isEditMode.value) return;

  console.log(
    `[编辑模式] 奖品 "${changedPrize.name}" 位置变更为: ${changedPrize.index}`
  );

  // 检查是否有其他奖品使用了相同的位置
  const conflictIndex = tempPrizeList.value.findIndex((prize, index) => {
    return prize.index === changedPrize.index && index !== changedIndex;
  });

  if (conflictIndex !== -1) {
    // 找到冲突的奖品，执行位置值交换（而不是数组元素交换）
    const conflictPrize = tempPrizeList.value[conflictIndex];
    const originalPosition = tempPrizeList.value[changedIndex].index;

    console.log(
      `[编辑模式] 检测到位置冲突: "${changedPrize.name}"(当前位置${originalPosition}) 与 "${conflictPrize.name}"(位置${changedPrize.index}) 需要交换位置值`
    );

    // 交换两个奖品的位置值
    const tempValue = tempPrizeList.value[changedIndex];
    tempPrizeList.value[changedIndex] = tempPrizeList.value[conflictIndex];
    tempPrizeList.value[conflictIndex] = tempValue;

    console.log(
      `[编辑模式] 位置值交换完成: "${changedPrize.name}"现在位置是${tempPrizeList.value[changedIndex].index}, "${conflictPrize.name}"现在位置是${tempPrizeList.value[conflictIndex].index}`
    );

    // 触发动画效果
    const table = document.querySelector(".prize-table");
    if (table) {
      table.classList.add("swapping");
      setTimeout(() => {
        if (table) {
          table.classList.remove("swapping");
        }
      }, 300);
    }

    ElMessage.success(
      `位置交换成功！"${conflictPrize.name}"与"${changedPrize.name}"已交换位置`
    );
  } else {
    // 没有冲突，正常移动
    ElMessage.success(
      `"${changedPrize.name}"已成功移动到位置${changedPrize.index}`
    );
  }

  console.log(
    "[编辑模式] 当前临时奖品列表位置分布:",
    tempPrizeList.value.map((p) => ({
      name: p.name,
      index: p.index,
    }))
  );
};
</script>

<style scoped>
/* 表格交换动画 */
.prize-table.swapping .el-table__body-wrapper {
  transition: transform 0.3s ease-in-out;
}

.prize-row {
  transition: all 0.3s ease-in-out;
}

.prize-table.swapping .prize-row {
  animation: slideDown 0.3s ease-in-out;
}

@keyframes slideDown {
  0% {
    transform: translateY(0);
    opacity: 1;
  }
  50% {
    transform: translateY(10px);
    opacity: 0.7;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

/* 行高亮效果 */
.prize-table .el-table__row:hover {
  background-color: #f5f7fa;
  transition: background-color 0.2s ease;
}

/* 交换时的特殊效果 */
.prize-table.swapping .el-table__row {
  background-color: #e6f7ff;
}
</style>
