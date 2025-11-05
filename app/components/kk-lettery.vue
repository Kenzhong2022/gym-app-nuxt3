<template>
  <!-- 主要内容区域 -->
  <div
    class="bg-white rounded-2xl sm:rounded-3xl shadow-2xl p-4 sm:p-6 md:p-8 w-full max-w-[95vw] sm:max-w-lg mx-auto"
  >
    <h1
      class="text-xl sm:text-2xl md:text-3xl font-bold text-center mb-4 sm:mb-6 md:mb-8 text-gray-800"
    >
      九宫格抽奖
    </h1>

    <!-- 九宫格容器 -->
    <div
      class="grid grid-cols-3 gap-20px md:gap-30px w-full max-w-[80vw] sm:max-w-[400px] mx-auto aspect-square"
    >
      <div v-for="pos in 9" :key="pos" class="relative">
        <div
          v-if="pos === 5"
          @click="startLottery"
          class="rounded-lg sm:rounded-xl shadow-lg w-full h-full flex items-center justify-center text-white font-bold text-sm sm:text-base md:text-xl cursor-pointer transition-all duration-200 hover:scale-105 active:scale-95 select-none"
          :class="
            isRunning
              ? 'bg-gray-400 cursor-not-allowed'
              : 'bg-gradient-to-br from-purple-500 to-pink-500'
          "
        >
          开始抽奖
        </div>
        <div
          v-else
          class="rounded-lg sm:rounded-xl shadow-lg w-full h-full flex items-center justify-center text-white font-bold text-xs sm:text-sm md:text-base transition-all duration-300 text-center select-none"
          :class="
            currentPosition === getPrizeNameByPosition(pos).id
              ? 'bg-red-500 scale-110 shadow-xl'
              : 'bg-gradient-to-br from-orange-400 to-red-500 hover:scale-105'
          "
        >
          <span class="px-1 leading-tight">{{
            getPrizeNameByPosition(pos).name
          }}</span>
        </div>
      </div>
    </div>

    <!-- 结果显示 -->
    <div
      v-if="finalResult"
      class="mt-4 sm:mt-6 p-3 sm:p-4 bg-green-100 rounded-lg text-center"
    >
      <p class="text-green-800 font-bold text-base sm:text-lg">
        🎉 恭喜获得: {{ finalResult }}
      </p>
    </div>

    <!-- 说明 -->
    <div class="mt-4 sm:mt-6 text-center text-gray-600 text-xs sm:text-sm">
      <p class="mt-1 sm:mt-2">点击中心"开始"按钮开始抽奖</p>
    </div>
  </div>
</template>

<script setup>
/* ==========  事件声明  ========== */
// 父组件可监听 @lottery-start 与 @lottery-end
const emit = defineEmits(["lotteryStart", "lotteryEnd"]);

/* ==========  Props定义  ========== */
const props = defineProps({
  list: {
    type: Array,
    default: () => [],
  },
  isRandom: {
    type: Boolean,
    default: true,
  },
});

/* ==========  响应式状态  ========== */
const isRunning = ref(false); // 抽奖是否正在进行
const currentPosition = ref(-1); // 当前高亮格子的「数组下标」；-1 表示全灭
const finalResult = ref(""); // 抽中的奖品文字

/* ==========  调试日志：看高亮走到哪一格  ========== */
watch(currentPosition, (newPos) => {
  if (newPos === -1) {
    console.log("[九宫格] 位置重置，所有格子熄灭");
  } else {
    const visualNumber = visual2Index.indexOf(newPos) + 1; // 视觉编号 1~8
    console.log(
      `[九宫格] 高亮 → 数组下标 ${newPos}（视觉编号 ${visualNumber}）`
    );
  }
});

/* ==========  核心配置  ========== */
/**
 * “九宫格” 8 个奖品格按从左到右、从上到下跳过中心按钮的顺序，依次对应后端 prizes 数组的第 0~7 号元素：
 * 1 号格 → prizes[0]
 * 2 号格 → prizes[1]
 * 3 号格 → prizes[2]
 * 4 号格 → prizes[7]
 * 6 号格 → prizes[3]
 * 7 号格 → prizes[6]
 * 8 号格 → prizes[5]
 * 9 号格 → prizes[4]
 */
const visual2Index = [1, 2, 3, 4, 5, 6, 7, 8];

// 计算奖品列表
const prizes = computed(() => {
  if (props.list && props.list.length > 0) {
    // 根据index属性重新排序奖品，确保它们在正确的位置
    const sortedPrizes = new Array(9).fill(null);
    props.list.forEach((item) => {
      if (item.index !== undefined && item.index >= 0 && item.index <= 8) {
        sortedPrizes[item.index] = item.name;
      }
    });
    // 填充空位置为默认奖品
    for (let i = 0; i < 9; i++) {
      if (i !== 4 && !sortedPrizes[i]) {
        // 跳过中心位置4
        sortedPrizes[i] = `奖品${i + 1}`;
      }
    }
    return sortedPrizes.filter((_, index) => index !== 4); // 排除中心位置
  }
  // 默认奖品（按正确顺序）
  return [
    "一等奖 🏆", // index 0
    "二等奖 🥈", // index 1
    "三等奖 🥉", // index 2
    "四等奖 🎁", // index 3
    "五等奖 🎈", // index 5 (跳过4)
    "六等奖 🎊", // index 6
    "七等奖 🎉", // index 7
    "谢谢参与 💝", // index 8
  ];
});

/* ==========  监听奖品列表变化  ========== */
watch(
  prizes,
  (newPrizes, oldPrizes) => {
    console.log("[奖品列表] 奖品列表发生变化:");
    console.log("[奖品列表] 旧奖品列表:", oldPrizes);
    console.log("[奖品列表] 新奖品列表:", newPrizes);
    console.log("[奖品列表] 奖品数量:", newPrizes.length);

    if (props.list && props.list.length > 0) {
      console.log("[奖品列表] 来自父组件的奖品数据:");
      props.list.forEach((item, index) => {
        console.log(
          `[奖品列表] 奖品${index + 1}: ID=${item.id}, 名称=${
            item.name
          }, 数组下标=${item.index}, 必中=${item.required || false}`
        );
      });
    }
  },
  { deep: true }
);

/* ==========  小工具：延迟函数  ========== */
const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

/* ==========  抽奖主流程 - 算法核心  ========== */
const startLottery = async () => {
  if (isRunning.value) return; // 防重复点击
  isRunning.value = true; // 上锁
  finalResult.value = ""; // 清空旧结果
  currentPosition.value = -1; // 熄灭所有灯

  emit("lotteryStart"); // 通知父组件"我开始了"

  let targetIndex;

  /* ===== 必中奖品算法核心 =====
   * 1. 检查isRandom参数：false表示启用必中逻辑
   * 2. 遍历奖品列表，查找required为true的奖品
   * 3. 如果找到必中奖品，确定其在列表中的索引位置
   * 4. 如果没有必中奖品，退回到随机选择模式
   * 5. 确保每次抽奖只有一个奖品能被必中
   */
  if (!props.isRandom && props.list && props.list.length > 0) {
    const requiredPrizes = props.list.filter((item) => item.required);
    if (requiredPrizes.length > 0) {
      // 找到必中奖品的索引
      const requiredPrize = requiredPrizes[0];
      targetIndex = props.list.findIndex(
        (item) => item.id === requiredPrize.id
      );
      console.log(
        `[抽奖] 检测到必中奖品: ${requiredPrize.name}, 目标索引: ${targetIndex}`
      );
    }
  }

  // 如果没有指定必中奖品，则随机选择
  if (targetIndex === undefined) {
    targetIndex = Math.floor(Math.random() * prizes.value.length);
    console.log(`[抽奖] 随机选择奖品, 目标索引: ${targetIndex}`);
  }

  /* ===== 转动步数算法核心 =====
   * 1. 基础圈数：3-4圈，营造转动的视觉效果
   * 2. 目标位置计算：根据targetIndex确定最终停止位置
   * 3. 步数计算：总步数 = 圈数 * 8 + 到目标的步数
   * 4. 确保每次都能准确停在目标奖品位置
   *
   * 移动路径: 0→1→2→5→8→7→6→3→0 (循环)
   * 例如targetIndex=2表示要停在第3个奖品位置
   */
  const rounds = 3 + Math.floor(Math.random() * 2); // 3 或 4圈基础转动
  const currentPos = 0; // 从位置0开始
  const targetPos = targetIndex; // 目标位置
  let stepsToTarget;

  // 计算从当前位置到目标位置的最短步数
  if (targetPos >= currentPos) {
    stepsToTarget = targetPos - currentPos;
  } else {
    stepsToTarget = visual2Index.length - currentPos + targetPos;
  }

  const totalSteps = rounds * visual2Index.length + stepsToTarget;
  console.log(
    `[抽奖] 总步数: ${totalSteps} (圈数: ${rounds}, 到目标步数: ${stepsToTarget})`
  );

  /* ===== 动画效果算法核心 =====
   * 1. 遍历每一步，计算当前应该高亮的格子位置
   * 2. 使用模运算确保位置在0-7范围内循环
   * 3. 通过visual2Index映射到实际的DOM元素索引
   * 4. 每一步延迟时间逐渐增大，实现减速效果
   * 5. 最终精确停在目标奖品对应的格子
   */
  for (let i = 0; i <= totalSteps; i++) {
    const pos = i % visual2Index.length; // 当前在8格中的序号
    currentPosition.value = visual2Index[pos]; // 点亮对应格子
    await sleep(Math.max(100, 300 - i * 8)); // 越跑越慢，营造减速效果
  }

  /* ===== 结果处理算法核心 =====
   * 1. 根据targetIndex获取最终奖品名称（而非根据停止位置）
   * 2. 确保结果与目标奖品完全一致
   * 3. 触发结束事件，通知父组件抽奖结果
   * 4. 重置状态，准备下一次抽奖
   */
  finalResult.value = prizes.value[targetIndex]; // 使用计算后的目标奖品
  console.log(`[抽奖] 最终结果: ${finalResult.value}`);
  emit("lotteryEnd", finalResult.value); // 通知父组件"我结束了，这是奖品"
  isRunning.value = false; // 解锁
};

/* ==========  父组件可手动调用：重置状态  ========== */
const resetLottery = () => {
  isRunning.value = false;
  currentPosition.value = -1;
  finalResult.value = "";
};

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

  if (prizeIndex !== undefined && props.list[prizeIndex]) {
    return props.list[prizeIndex];
  }
  return "空";
};

/* ==========  把方法/状态暴露给父组件  ========== */
defineExpose({
  startLottery,
  resetLottery,
  isRunning,
  currentPosition,
  finalResult,
});
</script>
