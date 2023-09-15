<script setup lang="ts">
import { getParkInfo } from '@/api/park'
import type { ParkResponseType } from '@/types/park'
import { onMounted, ref } from 'vue'
//引入echarts
import * as echarts from 'echarts'
import { Application } from '@splinetool/runtime'
//保存柱状图dom节点
const barChart = ref(null)
//保存饼状图数据
const pieChart = ref(null)
//保存园区概况得数据-产业分布
const parkInfo = ref<ParkResponseType>()
//调用接口
const initParkInfo = async () => {
  const res = await getParkInfo()
  console.log(res)
  parkInfo.value = res.data
}
//渲染echarts图表
//1.下载并安装
//2.获取渲染dom容器
const initBarChart = () => {
  //获取园区柱状图数据
  const { parkIncome } = parkInfo.value!
  console.log(parkIncome)

  //3.初始化echarts并指定渲染容器
  const myPieChart = echarts.init(barChart.value)
  //4.设置配置项
  const barOption = {
    tooltip: {
      show: true,
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      }
    },
    grid: {
      top: '10px',
      left: '0px',
      right: '0px',
      bottom: '0px',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: parkIncome.xMonth,
      axisTick: {
        show: false,
        alignWithLabel: true
      }
    },
    yAxis: {
      type: 'value',
      splitLine: {
        show: false
      }
    },
    textStyle: {
      color: '#b4c0cc'
    },
    series: [
      {
        data: parkIncome.yIncome.map((item: any, index: number) => {
          const color =
            index % 2 === 0
              ? {
                  type: 'linear',
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [
                    {
                      offset: 0,
                      color: '#74c0f8' // 0% 处的颜色
                    },
                    {
                      offset: 1,
                      color: 'rgba(116,192,248,0)' // 100% 处的颜色
                    }
                  ],
                  global: false // 缺省为 false
                }
              : {
                  type: 'linear',
                  x: 0,
                  y: 0,
                  x2: 0,
                  y2: 1,
                  colorStops: [
                    {
                      offset: 0,
                      color: '#ff7152' // 0% 处的颜色
                    },
                    {
                      offset: 1,
                      color: 'rgba(255,113,82,0)' // 100% 处的颜色
                    }
                  ],
                  global: false // 缺省为 false
                }
          return { value: item, itemStyle: { color } }
        }),
        type: 'bar',
        showBackground: true,
        backgroundStyle: {
          color: 'rgba(180, 180, 180, 0.2)'
        },
        barWidth: '10px'
      }
    ]
  }
  //5.渲染图表
  myPieChart && myPieChart.setOption(barOption)
  //6.设置图表自适应
}

const initPieChart = () => {
  //1.获取渲染数据
  const { parkIndustry } = parkInfo.value!

  //2.实例化echarts并指定渲染的dom容器
  const myPieChart = echarts.init(pieChart.value)

  //3.创建配置项
  const option = {
    color: ['#00B2FF', '#2CF2FF', '#892CFF', '#FF624D', '#FFCF54', '#86ECA2'],
    tooltip: {
      trigger: 'item'
    },
    legend: {
      itemGap: 10,
      bottom: '0',
      left: 'center',
      icon: 'rect',
      itemWidth: 10,
      itemHeight: 10,
      textStyle: {
        color: '#c6d1db'
      }
    },
    series: [
      {
        tooltip: {
          trigger: 'item',
          formatter: (params: any) => {
            console.log('params', params)
            return `${params.seriesName}<br/>${params.marker}${params.name}${params.percent}%`
          }
        },
        name: '园区产业分析',
        type: 'pie',
        radius: ['65%', '70%'],
        center: ['50%', '36%'],
        avoidLabelOverlap: false,
        label: {
          show: false,
          position: 'center'
        },
        labelLine: {
          show: false
        },
        data: parkIndustry
      }
    ]
  }
  //4.渲染配置项
  myPieChart && myPieChart.setOption(option)
  //5.图表自适应
}

onMounted(async () => {
  init3dModel()
  console.log(barChart.value)
  await initParkInfo()
  initBarChart()
  initPieChart()
})

//3d模型
const ref3d = ref()
const loading = ref(false)
const init3dModel = () => {
  loading.value = true
  console.log('ref3d', ref3d)
  const spline = new Application(ref3d.value)
  spline.load('https://fe-hmzs.itheima.net/scene.splinecode').then((res) => {
    console.log('模型加载完毕之后会触发then方法')
    loading.value = false
  })
}
</script>
<template>
  <div class="all-charts">
    <!-- 园区概况 -->
    <div class="section-one">
      <img
        class="img-header"
        src="https://yjy-teach-oss.oss-cn-beijing.aliyuncs.com/smartPark/%E5%A4%A7%E5%B1%8F%E5%88%87%E5%9B%BE/%E5%9B%AD%E5%8C%BA%E6%A6%82%E5%86%B5%402x.png"
        alt="logo"
      />
      <div class="icon-container">
        <div class="item">
          <div class="icons-item building-icon">
            <span class="number"> {{ parkInfo?.base.buildingTotal }} </span>
          </div>
          <span class="title">楼宇总数</span>
          <span class="untiy">(栋)</span>
        </div>
        <div class="item">
          <div class="icons-item enterprise-icon">
            <span class="number"> {{ parkInfo?.base.enterpriseTotal }} </span>
          </div>
          <span class="title">入驻企业总数</span>
          <span class="untiy">(栋)</span>
        </div>
        <div class="item">
          <div class="icons-item rod-icon">
            <span class="number"> {{ parkInfo?.base.parkingTotal }} </span>
          </div>
          <span class="title">车位总数</span>
          <span class="untiy">(栋)</span>
        </div>
        <div class="item">
          <div class="icons-item car-icon">
            <span class="number"> {{ parkInfo?.base.chargePoleTotal }} </span>
          </div>
          <span class="title">一体杆总数</span>
          <span class="untiy">(栋)</span>
        </div>
      </div>
    </div>
    <!-- 园区年度收入分析 -->
    <div class="section-two">
      <img
        class="img-header"
        src="https://yjy-teach-oss.oss-cn-beijing.aliyuncs.com/smartPark/%E5%A4%A7%E5%B1%8F%E5%88%87%E5%9B%BE/%E5%9B%AD%E5%8C%BA%E5%B9%B4%E5%BA%A6%E6%94%B6%E5%85%A5%E5%88%86%E6%9E%90%402x.png"
        alt="logo"
      />
      <div class="bar-chart-title">
        <span>单位:元</span>
        <div>
          <span class="bar-icon blue-bar-icon"></span>
          <span class="bar-icon red-bar-icon"></span>
          收入情况
        </div>
      </div>
      <div class="bar-chart" ref="barChart"></div>
    </div>
    <!-- 园区产业分布 -->
    <div class="section-three">
      <img
        class="img-header"
        src="https://yjy-teach-oss.oss-cn-beijing.aliyuncs.com/smartPark/%E5%A4%A7%E5%B1%8F%E5%88%87%E5%9B%BE/%E5%9B%AD%E5%8C%BA%E4%BA%A7%E4%B8%9A%E5%88%86%E5%B8%83%402x.png"
        alt="logo"
      />
      <div class="pie-chart" ref="pieChart"></div>
    </div>
  </div>
  <div class="model-container">
    <canvas
      style="width: 100%; height: 100%"
      class="canvas-3d"
      ref="ref3d"
    ></canvas>
    <!-- 加载进度条 -->
    <LoadingComponent :loading="loading"></LoadingComponent>
  </div>
</template>
<style lang="scss" scoped>
.all-charts {
  position: absolute;
  left: 0;
  top: 0;
  width: 480px;
  height: 100vh;
  padding: 20px;
  background: linear-gradient(
    to left,
    rgba(0, 6, 15, 0) 0%,
    rgba(0, 6, 15, 0) 20%,
    rgba(0, 0, 0, 0.4) 30%,
    rgba(0, 0, 0, 0.6) 40%,
    rgba(1, 4, 11, 1) 70%,
    #04070d 100%
  );
  .img-header {
    height: 30px;
  }
  .section-one {
    // flex-basis: 25%;
    .icon-container {
      display: flex;
      justify-content: space-between;
      padding: 20px 0;
      .item {
        text-align: center;
        display: flex;
        flex: 1;
        flex-direction: column;
        .icons-item {
          height: 80px;
          position: relative;
          .number {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            font-size: 18px;
            color: #fff;
          }
        }
        .building-icon {
          background: url('@/assets/building-icon.png') no-repeat 50% 0 /
            contain;
        }
        .enterprise-icon {
          background: url('@/assets/enterprise-icon.png') no-repeat 50% 0 /
            contain;
        }
        .rod-icon {
          background: url('@/assets/rod-icon.png') no-repeat 50% 0 / contain;
        }
        .car-icon {
          background: url('@/assets/car-icon.png') no-repeat 50% 0 / contain;
        }
        .title {
          margin-top: 8px;
        }

        .title,
        .untiy {
          font-size: 14px;
          color: #cdd7e1;
        }
      }
    }
  }
  .section-two {
    flex-basis: 35%;
    margin-top: 20px;

    .bar-chart-title {
      display: flex;
      color: #c6d1db;
      align-items: center;
      justify-content: space-between;
      font-size: 14px;
      margin-top: 20px;
      // height: 40px;

      .bar-icon {
        display: inline-block;
        width: 12px;
        height: 4px;
      }

      .blue-bar-icon {
        background: linear-gradient(90deg, #74c0f8, rgba(116, 192, 248, 0));
      }
      .red-bar-icon {
        background: linear-gradient(90deg, #ff7152, rgba(255, 113, 82, 0));
      }
    }

    .bar-chart {
      width: 100%;
      height: 200px;
    }
  }
  .section-three {
    flex-basis: 40%;
    margin-top: 40px;
    .pie-chart {
      width: 80%;
      height: 200px;
      margin: 0 auto;
      // padding-bottom: 20px;
    }
  }
}
.model-container {
  width: 100%;
  height: 100%;
  background-color: black;
}
</style>
