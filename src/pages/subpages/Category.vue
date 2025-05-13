<template>
    <div class="detail-container">
      <!-- 加载状态 -->
      <div v-if="loading" class="loading">加载中...</div>
  
      <!-- 建筑选择器 -->
      <div v-if="!loading" class="building-selector">
        <el-radio-group v-model="selectedBuilding" @change="handleBuildingChange">
          <el-radio-button label="A">A栋教学楼</el-radio-button>
          <el-radio-button label="B">B栋教学楼</el-radio-button>
          <el-radio-button label="C">C栋教学楼</el-radio-button>
          <el-radio-button label="D">D栋教学楼</el-radio-button>
        </el-radio-group>
      </div>
  
      <!-- 总体统计 -->
      <div v-if="!loading" class="summary-section">
        <h2>四栋教学楼总能耗</h2>
        <div class="summary-stats">
          <div class="summary-item">
            <div class="summary-label">总用电量</div>
            <div class="summary-value">{{ totalElectricity }} kWh</div>
            <div class="summary-compare">
              占全校用电 {{ totalElectricityPercent }}%
            </div>
          </div>
          <div class="summary-item">
            <div class="summary-label">总用水量</div>
            <div class="summary-value">{{ totalWater }} 吨</div>
            <div class="summary-compare">占全校用水 {{ totalWaterPercent }}%</div>
          </div>
          <div class="summary-item">
            <div class="summary-label">能耗指数</div>
            <div class="summary-value">{{ energyIndex }}</div>
            <div class="summary-compare">较上月 {{ energyIndexTrend }}%</div>
          </div>
        </div>
      </div>
  
      <!-- 基础信息展示 -->
      <div v-if="!loading" class="building-info">
        <h2>{{ currentBuildingData.name }} 能耗详情</h2>
        <div class="stats">
          <div class="stat-item">
            <label>今日用电</label>
            <div class="value">{{ currentBuildingData.day }} kWh</div>
            <div
              class="trend"
              :class="getTrendClass(currentBuildingData.dayTrend)"
            >
              <i :class="getTrendIcon(currentBuildingData.dayTrend)"></i>
              {{ currentBuildingData.dayTrend }}%
            </div>
          </div>
          <div class="stat-item">
            <label>本月用电</label>
            <div class="value">{{ currentBuildingData.month }} kWh</div>
            <div
              class="trend"
              :class="getTrendClass(currentBuildingData.monthTrend)"
            >
              <i :class="getTrendIcon(currentBuildingData.monthTrend)"></i>
              {{ currentBuildingData.monthTrend }}%
            </div>
          </div>
          <div class="stat-item">
            <label>今日用水</label>
            <div class="value">{{ currentBuildingData.waterDay }} 吨</div>
            <div
              class="trend"
              :class="getTrendClass(currentBuildingData.waterDayTrend)"
            >
              <i :class="getTrendIcon(currentBuildingData.waterDayTrend)"></i>
              {{ currentBuildingData.waterDayTrend }}%
            </div>
          </div>
          <div class="stat-item">
            <label>本月用水</label>
            <div class="value">{{ currentBuildingData.waterMonth }} 吨</div>
            <div
              class="trend"
              :class="getTrendClass(currentBuildingData.waterMonthTrend)"
            >
              <i :class="getTrendIcon(currentBuildingData.waterMonthTrend)"></i>
              {{ currentBuildingData.waterMonthTrend }}%
            </div>
          </div>
          <div class="stat-item">
            <label>能耗排名</label>
            <div class="value">{{ currentBuildingData.rank }}/4</div>
            <div class="trend">
              <i
                :class="
                  currentBuildingData.rankChange > 0
                    ? 'el-icon-top'
                    : currentBuildingData.rankChange < 0
                    ? 'el-icon-bottom'
                    : 'el-icon-minus'
                "
              ></i>
              {{ Math.abs(currentBuildingData.rankChange) || "无" }}
            </div>
          </div>
        </div>
      </div>
  
      <!-- 单个建筑图表 -->
      <div v-show="!loading" class="chart-section">
        <h3>{{ currentBuildingData.name }} 近7天能耗趋势</h3>
        <el-tabs type="border-card">
          <el-tab-pane label="用电趋势">
            <div ref="singleElectricChartEl" class="chart-box"></div>
          </el-tab-pane>
          <el-tab-pane label="用水趋势">
            <div ref="singleWaterChartEl" class="chart-box"></div>
          </el-tab-pane>
        </el-tabs>
      </div>
  
      <!-- 四栋楼对比图表 -->
      <div v-show="!loading" class="chart-section">
        <h3>四栋教学楼本月能耗对比</h3>
        <el-tabs type="border-card">
          <el-tab-pane label="用电对比">
            <div ref="compareElectricChartEl" class="chart-box"></div>
          </el-tab-pane>
          <el-tab-pane label="用水对比">
            <div ref="compareWaterChartEl" class="chart-box"></div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, onBeforeUnmount } from "vue";
  import * as echarts from "echarts";
  
  // 图表实例
  const singleElectricChartEl = ref(null);
  const singleWaterChartEl = ref(null);
  const compareElectricChartEl = ref(null);
  const compareWaterChartEl = ref(null);
  let singleElectricChart = null;
  let singleWaterChart = null;
  let compareElectricChart = null;
  let compareWaterChart = null;
  
  // 响应式数据
  const loading = ref(true);
  const selectedBuilding = ref("A");
  const buildingsData = ref({
    A: {
      name: "A栋教学楼",
      day: 245.8,
      month: 6543.2,
      dayTrend: 12.5,
      monthTrend: 8.3,
      waterDay: 56.2,
      waterMonth: 1680.5,
      waterDayTrend: 5.3,
      waterMonthTrend: 3.2,
      rank: 2,
      rankChange: 1,
      weeklyElectricData: [320, 332, 301, 334, 390, 330, 320],
      weeklyWaterData: [45, 52, 48, 55, 60, 58, 56],
    },
    B: {
      name: "B栋教学楼",
      day: 312.4,
      month: 7321.5,
      dayTrend: -5.2,
      monthTrend: 12.1,
      waterDay: 72.8,
      waterMonth: 1920.3,
      waterDayTrend: -3.7,
      waterMonthTrend: 8.5,
      rank: 1,
      rankChange: 0,
      weeklyElectricData: [420, 432, 401, 434, 490, 430, 420],
      weeklyWaterData: [68, 72, 65, 70, 75, 73, 72],
    },
    C: {
      name: "C栋教学楼",
      day: 198.6,
      month: 5210.7,
      dayTrend: 3.7,
      monthTrend: -2.4,
      waterDay: 42.5,
      waterMonth: 1250.8,
      waterDayTrend: 8.2,
      waterMonthTrend: -5.1,
      rank: 3,
      rankChange: -1,
      weeklyElectricData: [220, 232, 201, 234, 290, 230, 220],
      weeklyWaterData: [38, 42, 40, 45, 50, 48, 42],
    },
    D: {
      name: "D栋教学楼",
      day: 156.3,
      month: 4876.9,
      dayTrend: -8.1,
      monthTrend: -5.6,
      waterDay: 35.7,
      waterMonth: 980.2,
      waterDayTrend: -10.5,
      waterMonthTrend: -8.3,
      rank: 4,
      rankChange: 0,
      weeklyElectricData: [180, 192, 171, 194, 190, 230, 210],
      weeklyWaterData: [32, 35, 30, 36, 38, 35, 35],
    },
  });
  
  // 计算属性
  const currentBuildingData = computed(
    () => buildingsData.value[selectedBuilding.value]
  );
  const totalElectricity = computed(() =>
    Object.values(buildingsData.value)
      .reduce((sum, b) => sum + b.month, 0)
      .toFixed(1)
  );
  const totalWater = computed(() =>
    Object.values(buildingsData.value)
      .reduce((sum, b) => sum + b.waterMonth, 0)
      .toFixed(1)
  );
  const totalElectricityPercent = computed(() =>
    ((totalElectricity.value / 25000) * 100).toFixed(1)
  );
  const totalWaterPercent = computed(() =>
    ((totalWater.value / 7000) * 100).toFixed(1)
  );
  const energyIndex = computed(() =>
    (totalElectricity.value / 1000 + totalWater.value / 100).toFixed(2)
  );
  const energyIndexTrend = computed(() => (Math.random() * 5 - 2).toFixed(1));
  
  // 初始化单个建筑用电图表
  const initSingleElectricChart = () => {
    if (!singleElectricChartEl.value) return;
  
    if (singleElectricChart) singleElectricChart.dispose();
  
    singleElectricChart = echarts.init(singleElectricChartEl.value);
  
    const option = {
      tooltip: {
        trigger: "axis",
        formatter: "{b}<br/>{a}: {c} kWh",
      },
      xAxis: {
        type: "category",
        data: ["周一", "周二", "周三", "周四", "周五", "周六", "周日"],
      },
      yAxis: {
        type: "value",
        name: "用电量(kWh)",
      },
      series: [
        {
          name: "用电量",
          data: currentBuildingData.value.weeklyElectricData,
          type: "line",
          smooth: true,
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: "rgba(58, 77, 233, 0.8)" },
              { offset: 1, color: "rgba(58, 77, 233, 0.1)" },
            ]),
          },
          itemStyle: {
            color: "#3a4de9",
          },
          lineStyle: {
            width: 3,
          },
        },
      ],
    };
  
    singleElectricChart.setOption(option);
  };
  
  // 初始化单个建筑用水图表
  const initSingleWaterChart = () => {
    if (!singleWaterChartEl.value) return;
  
    if (singleWaterChart) singleWaterChart.dispose();
  
    singleWaterChart = echarts.init(singleWaterChartEl.value);
  
    const option = {
      tooltip: {
        trigger: "axis",
        formatter: "{b}<br/>{a}: {c} 吨",
      },
      xAxis: {
        type: "category",
        data: ["周一", "周二", "周三", "周四", "周五", "周六", "周日"],
      },
      yAxis: {
        type: "value",
        name: "用水量(吨)",
      },
      series: [
        {
          name: "用水量",
          data: currentBuildingData.value.weeklyWaterData,
          type: "line",
          smooth: true,
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: "rgba(86, 204, 162, 0.8)" },
              { offset: 1, color: "rgba(86, 204, 162, 0.1)" },
            ]),
          },
          itemStyle: {
            color: "#56cca2",
          },
          lineStyle: {
            width: 3,
          },
        },
      ],
    };
  
    singleWaterChart.setOption(option);
  };
  
  // 初始化四栋楼用电对比图表
  const initCompareElectricChart = () => {
    if (!compareElectricChartEl.value) return;
  
    if (compareElectricChart) compareElectricChart.dispose();
  
    compareElectricChart = echarts.init(compareElectricChartEl.value);
  
    const buildingNames = ["A栋", "B栋", "C栋", "D栋"];
    const monthData = [
      buildingsData.value.A.month,
      buildingsData.value.B.month,
      buildingsData.value.C.month,
      buildingsData.value.D.month,
    ];
  
    const option = {
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
        formatter: "{b}<br/>用电量: {c} kWh",
      },
      xAxis: {
        type: "category",
        data: buildingNames,
        axisLabel: {
          interval: 0,
          rotate: 0,
        },
      },
      yAxis: {
        type: "value",
        name: "用电量(kWh)",
      },
      series: [
        {
          name: "本月用电",
          type: "bar",
          data: monthData,
          itemStyle: {
            color: (params) => {
              const colors = ["#5470c6", "#91cc75", "#fac858", "#ee6666"];
              return colors[params.dataIndex];
            },
          },
          label: {
            show: true,
            position: "top",
            formatter: "{c} kWh",
          },
        },
      ],
    };
  
    compareElectricChart.setOption(option);
  };
  
  // 初始化四栋楼用水对比图表
  const initCompareWaterChart = () => {
    if (!compareWaterChartEl.value) return;
  
    if (compareWaterChart) compareWaterChart.dispose();
  
    compareWaterChart = echarts.init(compareWaterChartEl.value);
  
    const buildingNames = ["A栋", "B栋", "C栋", "D栋"];
    const monthData = [
      buildingsData.value.A.waterMonth,
      buildingsData.value.B.waterMonth,
      buildingsData.value.C.waterMonth,
      buildingsData.value.D.waterMonth,
    ];
  
    const option = {
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
        formatter: "{b}<br/>用水量: {c} 吨",
      },
      xAxis: {
        type: "category",
        data: buildingNames,
        axisLabel: {
          interval: 0,
          rotate: 0,
        },
      },
      yAxis: {
        type: "value",
        name: "用水量(吨)",
      },
      series: [
        {
          name: "本月用水",
          type: "bar",
          data: monthData,
          itemStyle: {
            color: (params) => {
              const colors = ["#73c0de", "#3ba272", "#fc8452", "#9a60b4"];
              return colors[params.dataIndex];
            },
          },
          label: {
            show: true,
            position: "top",
            formatter: "{c} 吨",
          },
        },
      ],
    };
  
    compareWaterChart.setOption(option);
  };
  
  // 处理建筑切换
  const handleBuildingChange = () => {
    initSingleElectricChart();
    initSingleWaterChart();
    window.dispatchEvent(new Event("resize"));
  };
  
  // 处理窗口变化
  const handleResize = () => {
    singleElectricChart?.resize();
    singleWaterChart?.resize();
    compareElectricChart?.resize();
    compareWaterChart?.resize();
  };
  
  // 获取趋势样式
  const getTrendClass = (trend) => {
    return trend >= 0 ? "up" : "down";
  };
  
  // 获取趋势图标
  const getTrendIcon = (trend) => {
    return trend >= 0 ? "el-icon-top" : "el-icon-bottom";
  };
  
  // 模拟数据请求
  const fetchData = () => {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(true);
      }, 800);
    });
  };
  
  // 生命周期
  onMounted(async () => {
    try {
      await fetchData();
  
      // 初始化所有图表
      initSingleElectricChart();
      initSingleWaterChart();
      initCompareElectricChart();
      initCompareWaterChart();
  
      window.addEventListener("resize", handleResize);
    } finally {
      loading.value = false;
    }
  });
  
  onBeforeUnmount(() => {
    // 清理所有图表实例
    [
      singleElectricChart,
      singleWaterChart,
      compareElectricChart,
      compareWaterChart,
    ].forEach((chart) => {
      if (chart) {
        chart.dispose();
        chart = null;
      }
    });
    window.removeEventListener("resize", handleResize);
  });
  </script>
  
  <style scoped>
  .detail-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }
  
  .loading {
    padding: 50px;
    text-align: center;
    color: #666;
  }
  
  .building-selector {
    margin-bottom: 20px;
    text-align: center;
  }
  
  .summary-section {
    margin-bottom: 30px;
    background: #f8f9fa;
    padding: 20px;
    border-radius: 8px;
  }
  
  .summary-section h2 {
    color: #333;
    margin-bottom: 20px;
    text-align: center;
  }
  
  .summary-stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
  }
  
  .summary-item {
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    text-align: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  }
  
  .summary-label {
    font-size: 16px;
    color: #666;
    margin-bottom: 10px;
  }
  
  .summary-value {
    font-size: 1.8em;
    color: #409eff;
    font-weight: bold;
    margin-bottom: 8px;
  }
  
  .summary-compare {
    font-size: 14px;
    color: #999;
  }
  
  .building-info {
    margin-bottom: 30px;
  }
  
  .building-info h2 {
    color: #333;
    margin-bottom: 20px;
    border-bottom: 1px solid #eee;
    padding-bottom: 10px;
  }
  
  .stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
  }
  
  .stat-item {
    background: #f8f9fa;
    padding: 20px;
    border-radius: 8px;
    position: relative;
  }
  
  .stat-item label {
    display: block;
    color: #666;
    margin-bottom: 8px;
    font-size: 14px;
  }
  
  .value {
    font-size: 1.4em;
    color: #409eff;
    font-weight: bold;
    margin-bottom: 5px;
  }
  
  .trend {
    font-size: 12px;
  }
  
  .trend.up {
    color: #f56c6c;
  }
  
  .trend.down {
    color: #67c23a;
  }
  
  .chart-section {
    margin-bottom: 40px;
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  }
  
  .chart-section h3 {
    margin-bottom: 20px;
    color: #333;
    font-weight: normal;
  }
  
  .chart-box {
    width: 100%;
    height: 350px;
  }
  
  @media (max-width: 768px) {
    .chart-box {
      height: 250px;
    }
  
    .stats,
    .summary-stats {
      grid-template-columns: 1fr;
    }
  }
  </style>
  