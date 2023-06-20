<template>
  <div class="chart-container">
    <div class="chart-wrapper">
      <div ref="tpsChartContainer" class="chart"></div>
    </div>
    <div class="chart-wrapper">
      <div ref="responseTimeChartContainer" class="chart"></div>
    </div>
    <div class="input-wrapper">
      <input v-model="transactionCount" type="number" placeholder="输入交易数量">
      <button @click="submitTransaction" :disabled="isRunning || submitting">提交</button>
      <button @click="executeTransaction" :disabled="isRunning || submitting">执行</button>
      <button @click="pauseExecution" :disabled="!timer">暂停</button>
      <p>{{ submitMessage }}</p>

    </div>
  </div>
</template>

<style>
.chart-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  align-items: flex-start;
}

.chart-wrapper {
  flex: 1 0 50%;
  margin-right: 20px;
}

.input-wrapper {
  margin-left: 600px;
  position: absolute; 
}

.chart {
  width: 500px;
  height: 300px;
}
</style>

<script>
import * as echarts from 'echarts'
import axios from 'axios'
import 'echarts/lib/chart/line'
import 'echarts/lib/component/dataZoom'
import 'echarts/lib/component/toolbox'


export default {
  data() {
    return {
      isRunning: false,
      submitting: false,
      submitMessage:' ',
      timer: null,
      transactionCount: 0,
      tpsData: [
       
      ],
      responseTimeData: [
        
      ]
    }
  },
  created() {
    this.fetchData()
  },
  mounted() {
    this.renderTPSChart()
    this.renderResponseTimeChart()
  },
  methods: {
    async fetchData() {
      // try {
      //   const res = await axios.get('http://150.158.173.17:3000/api/data')
      //   this.tpsData = res.data
      //   this.responseTimeData = res.data
      // } catch (err) {
      //   console.error(err)
      // }
    },
    renderTPSChart() {
      const chartContainer = this.$refs.tpsChartContainer
      const chart = echarts.init(chartContainer)

      const options = {
        grid: {
          containLabel: true
        },
        toolbox: {
          show: true,
          feature: {
            dataZoom: {
              show: true,
              title: {
                zoom: '区域缩放',
                back: '还原缩放'
              }
            },
            saveAsImage: {
              show: true,
              title: '保存为图片'
            }
          }
        },
        title: {
          text: 'TPS/s',
          left: 'center'
        },
        xAxis: {
          type: 'category',
          data: this.tpsData.map(item => item.time)
        },
        yAxis: {
          type: 'value',
          name: 'TPS'
        },
        series: [{
          type: 'line',
          data: this.tpsData.map(item => item.value),
          markPoint: {
            data: [
              { type: 'max', name: '最大值' },
              { type: 'min', name: '最小值' }
            ]
          }
        }]
      }

      chart.setOption(options)
    },
    renderResponseTimeChart() {
      const chartContainer = this.$refs.responseTimeChartContainer
      const chart = echarts.init(chartContainer)

      const options = {
        grid: {
          containLabel: true
        },
        toolbox: {
          show: true,
          feature: {
            dataZoom: {
              show: true,
              title: {
                zoom: '区域缩放',
                back: '还原缩放'
              }
            },
            saveAsImage: {
              show: true,
              title: '保存为图片'
            }
          }
        },
        title: {
          text: '响应时间',
          left: 'center'
        },
        xAxis: {
          type: 'category',
          data: this.responseTimeData.map(item => item.time)
        },
        yAxis: {
          type: 'value',
          name: '交易确认时间/ms'
        },
        series: [{
          type: 'line',
          data: this.responseTimeData.map(item => item.value),
          markPoint: {
            data: [
              { type: 'max', name: '最大值' },
              { type: 'min', name: '最小值' }
            ]
          }
        }]
      }

      chart.setOption(options)
    },
    executeTransaction() {
      this.isRunning = true;
      this.timer = setInterval(this.submitTransaction, 10000);
    },
    pauseExecution(){
      
      clearInterval(this.timer);
      this.timer = null;
      this.isRunning = false;
      this.submitting = false;
    },
    submitTransaction() {
  this.submitting = true;
  this.submitMessage = '正在提交交易....';
  console.log('提交交易数量:', this.transactionCount)

      setTimeout(() => {
            this.submitting = false;
            this.submitMessage = '';
            axios.post('http://150.158.173.17:3000/api/transaction', { count: this.transactionCount })
        .then(response => {
          // 处理成功响应
          console.log(response.data);
          let totalTps = 0;
          let totalDelay = 0;

          response.data.TpsResponse.forEach(tpsResponseStr => {
            let tpsResponse = JSON.parse(tpsResponseStr);
            totalTps += tpsResponse.tps;
            totalDelay += tpsResponse.averageDelay;
          });
          const avgDelay = totalDelay / response.data.TpsResponse.length;
          const currentTime = new Date().toLocaleString()

          // let tpsResponse = JSON.parse(response.data.TpsResponse);
          //解析TPS数据

          
          //添加响应时间数据
          this.responseTimeData.push({ time: currentTime, value: avgDelay }) 
          const responseTimeChart = echarts.getInstanceByDom(this.$refs.responseTimeChartContainer)
          responseTimeChart.setOption({
              xAxis: {
              data: this.responseTimeData.map(item => item.time)
              },
              series: [{
              data: this.responseTimeData.map(item => item.value)
        }]
      })

          this.tpsData.push({ time: currentTime, value: totalTps })
          const tpsChart = echarts.getInstanceByDom(this.$refs.tpsChartContainer)
          tpsChart.setOption({
            xAxis: {
              data: this.tpsData.map(item => item.time)
            },
            series: [{
              data: this.tpsData.map(item => item.value)
            }]
          })
        })
        .catch(error => {
          // 处理错误响应
          console.error('交易处理失败:', error);
        });
        
            }, 1000);
      // 发送交易的逻辑代码...
    }
  }
}
</script>
