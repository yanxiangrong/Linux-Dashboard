<template>
  <div style="display: flex; flex-direction: column">
    <div class="box">
      <div style="display: flex; flex-direction: column">
        <div style="display: flex; flex-direction: row">
          <div style="margin-top: -25px; text-align: center; width: 420px;">
            <div id="cpuChart" style="width: 420px;height:420px;"></div>
            <h2 style="margin-top: -116px">CPU</h2>
          </div>
          <div style="margin-top: -25px; text-align: center; width: 420px;">
            <div id="memChart" style="width: 420px;height:420px;"></div>
            <h2 style="margin-top: -116px">内存</h2>
          </div>
        </div>
        <div>
          <el-row :gutter="20">
            <el-col :span="4">
              <span>系统主机名</span>
            </el-col>
            <el-col :span="8">
              <span>{{ host.hostName }}</span>
            </el-col>
            <el-col :span="4">
              <span>操作系统</span>
            </el-col>
            <el-col :span="8">
              <span>{{ host.platform }}</span>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <span>系统时间</span>
            </el-col>
            <el-col :span="8">
              <span>todo</span>
            </el-col>
            <el-col :span="4">
              <span>内核</span>
            </el-col>
            <el-col :span="8">
              <span>{{ host.os }}, {{ host.kernelArch }}</span>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <span>CPU 信息</span>
            </el-col>
            <el-col :span="8">
              <span>{{ cpu.modelName }}, {{ cpu.cores }} 核心</span>
            </el-col>
            <el-col :span="4">
              <span>系统在线时间</span>
            </el-col>
            <el-col :span="8">
              <span>{{ host.uptime }}</span>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <span>正在运行的进程</span>
            </el-col>
            <el-col :span="8">
              <span>{{ host.process }}</span>
            </el-col>
            <el-col :span="4">
              <span>CPU 负载</span>
            </el-col>
            <el-col :span="8">
              <span>{{ cpu.load1 }} (1分钟) {{ cpu.load5 }} (5分钟) {{ cpu.load15 }} (15分钟)</span>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="4">
              <span>物理内存</span>
            </el-col>
            <el-col :span="8">
              <span>已用 {{ memory.used }} / 空闲 {{ memory.free }} / 总计 {{ memory.total }}</span>
            </el-col>
            <el-col :span="4">
              <span>本地磁盘空间</span>
            </el-col>
            <el-col :span="8">
              <span>已用 {{ disk.used }} / 空闲 {{ disk.free }} / 总计 {{ disk.total }}</span>
            </el-col>
          </el-row>
        </div>
      </div>
    </div>

    <div class="box">
      <div style="display: flex; flex-direction: row">
        <div>
          <div style="height: 420px; width: 630px" id="cpuHistory"></div>
        </div>
        <div>
          <div style="height: 420px; width: 630px" id="memHistory"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {defineComponent} from 'vue';
import * as echarts from 'echarts';
import axios from 'axios';

export default defineComponent({
  name: 'home',
  data() {
    return {
      cpu: {
        cores: 0,
        modelName: "unknown",
        load1: 0,
        load5: 0,
        load15: 0,
      },
      memory: {
        total: 0,
        available: 0,
        used: 0,
        free: 0
      },
      host: {
        hostName: "unknown",
        uptime: 0,
        process: 0,
        os: "unknown",
        kernelVersion: "unknown",
        kernelArch: "unknown",
        platform: "unknown",
        platformVersion: "unknown"
      },
      disk: {
        devices: "unknown",
        mountPoint: "unknown",
        fsType: "unknown",
        total: 0,
        used: 0,
        free: 0
      }
    }
  },
  methods: {
    updateDate() {
      const _this = this;
      const cpuChartElem = document.getElementById('cpuChart')!;
      const memChartElem = document.getElementById('memChart')!;
      const cpuHistoryElem = document.getElementById('cpuHistory')!;
      const memHistoryElem = document.getElementById('memHistory')!;
      const cpuChart = echarts.init(cpuChartElem);
      const memChart = echarts.init(memChartElem);
      const cpuHistory = echarts.init(cpuHistoryElem);
      const memHistory = echarts.init(memHistoryElem);

      setInterval(function () {
        let cpuUsage = 0;
        let memUsage = 0;
        axios.get('http://127.0.0.1:9527/v1/current')
            .then(function (response) {
              console.log(response.data);
              cpuUsage = response.data['cpu_usage_per'];
              memUsage = response.data['mem_usage_per'];
              cpuChart.setOption({
                series: [{
                  data: [{
                    value: cpuUsage
                  }]
                }]
              })
              memChart.setOption({
                series: [{
                  data: [{
                    value: memUsage
                  }]
                }]
              })
            })
      }, 2000)
      setInterval(function () {
        let cpuUsageList: any[][] = [];
        let memUsageList: any[][] = [];
        axios.get('http://127.0.0.1:9527/v1/history')
            .then(function (response) {
              // console.log(response.data);
              for (let i = 0; i < response.data.length; i++) {
                cpuUsageList[i] = [i, response.data[i]['cpu_usage_per']];
                memUsageList[i] = [i, response.data[i]['mem_usage_per']];
              }
              // console.log(cpuUsageList)
              cpuHistory.setOption({
                series: [{
                  data: cpuUsageList
                }]
              })
              memHistory.setOption({
                series: [{
                  data: memUsageList
                }]
              })
            })
      }, 2000)
      setInterval(function () {
        let cpuUsageList: any[][] = [];
        let memUsageList: any[][] = [];
        axios.get('http://127.0.0.1:9527/v1/moreInfo')
            .then(function (response) {
              _this.cpu.cores = response.data['cpu']['cores']
              _this.cpu.modelName = response.data['cpu']['model_name']
              _this.cpu.load1 = response.data['cpu']['load_1']
              _this.cpu.load5 = response.data['cpu']['load_5']
              _this.cpu.load15 = response.data['cpu']['load_15']
              _this.memory.total = response.data['Memory']['total']
              _this.memory.available = response.data['Memory']['available']
              _this.memory.used = response.data['Memory']['used']
              _this.memory.free = response.data['Memory']['free']
              _this.host.hostName = response.data['Host']['host_name']
              _this.host.uptime = response.data['Host']['uptime']
              _this.host.process = response.data['Host']['process']
              _this.host.os = response.data['Host']['os']
              _this.host.kernelVersion = response.data['Host']['kernel_version']
              _this.host.kernelArch = response.data['Host']['kernel_arch']
              _this.host.platform = response.data['Host']['platform']
              _this.host.platformVersion = response.data['Host']['platform_version']
              _this.disk.devices = response.data['Disks'][0]['devices']
              _this.disk.mountPoint = response.data['Disks'][0]['mount_point']
              _this.disk.fsType = response.data['Disks'][0]['fs_type']
              _this.disk.total = response.data['Disks'][0]['total']
              _this.disk.used = response.data['Disks'][0]['used']
              _this.disk.free = response.data['Disks'][0]['free']

            })
      }, 10000)
    },

    drawChart() {
      let cpuChart = echarts.init(document.getElementById('cpuChart')!);
      let memChart = echarts.init(document.getElementById('memChart')!);
      let cpuHistory = echarts.init(document.getElementById('cpuHistory')!);
      let memHistory = echarts.init(document.getElementById('memHistory')!);

      let option1 = {
        series: [{
          type: 'gauge',
          startAngle: 225,
          endAngle: -45,
          min: 0,
          max: 100,
          splitNumber: 10,
          itemStyle: {
            color: '#58D9F9',
            shadowColor: 'rgba(0,138,255,0.45)',
            shadowBlur: 10,
            shadowOffsetX: 2,
            shadowOffsetY: 2
          },
          progress: {
            show: true,
            roundCap: true,
            width: 14
          },
          pointer: {
            icon: 'path://M2090.36389,615.30999 L2090.36389,615.30999 C2091.48372,615.30999 2092.40383,616.194028 2092.44859,617.312956 L2096.90698,728.755929 C2097.05155,732.369577 2094.2393,735.416212 2090.62566,735.56078 C2090.53845,735.564269 2090.45117,735.566014 2090.36389,735.566014 L2090.36389,735.566014 C2086.74736,735.566014 2083.81557,732.63423 2083.81557,729.017692 C2083.81557,728.930412 2083.81732,728.84314 2083.82081,728.755929 L2088.2792,617.312956 C2088.32396,616.194028 2089.24407,615.30999 2090.36389,615.30999 Z',
            length: '65%',
            width: 12,
            offsetCenter: [0, '5%']
          },
          axisLine: {
            roundCap: true,
            lineStyle: {
              width: 14
            }
          },
          axisTick: {
            splitNumber: 2,
            lineStyle: {
              width: 1,
              color: '#999'
            }
          },
          splitLine: {
            length: 9,
            lineStyle: {
              width: 2,
              color: '#999'
            }
          },
          axisLabel: {
            distance: 22,
            color: '#999',
            fontSize: 15
          },
          title: {
            show: false,
            // text: 'CPU usage',
          },
          detail: {
            backgroundColor: '#fff',
            // borderColor: '#999',
            // borderWidth: 0,
            width: '60%',
            lineHeight: 40,
            height: 40,
            // borderRadius: 8,
            offsetCenter: [0, '35%'],
            valueAnimation: true,
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}';
            },
            rich: {
              value: {
                fontSize: 50,
                fontWeight: 'bolder',
                color: '#777'
              },
              unit: {
                fontSize: 20,
                color: '#999',
                padding: [0, 0, -20, 10]
              }
            }
          },
          data: [{
            value: 0
          }]
        }]
      };
      let option2 = {
        series: [{
          type: 'gauge',
          startAngle: 225,
          endAngle: -45,
          min: 0,
          max: 100,
          splitNumber: 10,
          itemStyle: {
            color: '#F97858',
            shadowColor: 'rgba(255,119,0,0.45)',
            shadowBlur: 10,
            shadowOffsetX: 2,
            shadowOffsetY: 2
          },
          progress: {
            show: true,
            roundCap: true,
            width: 14
          },
          pointer: {
            icon: 'path://M2090.36389,615.30999 L2090.36389,615.30999 C2091.48372,615.30999 2092.40383,616.194028 2092.44859,617.312956 L2096.90698,728.755929 C2097.05155,732.369577 2094.2393,735.416212 2090.62566,735.56078 C2090.53845,735.564269 2090.45117,735.566014 2090.36389,735.566014 L2090.36389,735.566014 C2086.74736,735.566014 2083.81557,732.63423 2083.81557,729.017692 C2083.81557,728.930412 2083.81732,728.84314 2083.82081,728.755929 L2088.2792,617.312956 C2088.32396,616.194028 2089.24407,615.30999 2090.36389,615.30999 Z',
            length: '65%',
            width: 12,
            offsetCenter: [0, '5%']
          },
          axisLine: {
            roundCap: true,
            lineStyle: {
              width: 14
            }
          },
          axisTick: {
            splitNumber: 2,
            lineStyle: {
              width: 1,
              color: '#999'
            }
          },
          splitLine: {
            length: 9,
            lineStyle: {
              width: 2,
              color: '#999'
            }
          },
          axisLabel: {
            distance: 22,
            color: '#999',
            fontSize: 15
          },
          title: {
            show: false,
            // text: 'CPU usage',
          },
          detail: {
            backgroundColor: '#fff',
            // borderColor: '#999',
            // borderWidth: 0,
            width: '60%',
            lineHeight: 40,
            height: 40,
            // borderRadius: 8,
            offsetCenter: [0, '35%'],
            valueAnimation: true,
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}';
            },
            rich: {
              value: {
                fontSize: 50,
                fontWeight: 'bolder',
                color: '#777'
              },
              unit: {
                fontSize: 20,
                color: '#999',
                padding: [0, 0, -20, 10]
              }
            }
          },
          data: [{
            value: 0
          }]
        }]
      };
      let option3 = {
        xAxis: {
          type: 'time',
          boundaryGap: false,
        },
        yAxis: {
          min: 0,
          max: 100,
          name: '60秒内的利用率%',
          type: 'value'
        },
        series: [{
          data: [
            [0, 0],
          ],
          type: 'line',
          areaStyle: {},
          symbol: 'none',
          animation: false
        }]
      };
      let option4 = {
        xAxis: {
          type: 'time',
          boundaryGap: false,
        },
        yAxis: {
          min: 0,
          max: 100,
          name: '内存使用量',
          type: 'value'
        },
        series: [{
          data: [
            [0, 0],
          ],
          type: 'line',
          areaStyle: {},
          symbol: 'none',
          animation: false
        }]
      };

      cpuChart.setOption(option1);
      memChart.setOption(option2);
      cpuHistory.setOption(option3);
      memHistory.setOption(option4);
    }
  },
  mounted() {
    this.updateDate();
    this.drawChart();
  }
})

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.box {
  margin: 8px 10px 2px 10px;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}
</style>
