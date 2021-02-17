<template>
  <div style="display: flex; flex-direction: column">
    <div class="box">
      <h3 class="panel-title">系统信息</h3>
      <el-divider></el-divider>
      <div style="display: flex; flex-direction: column">
        <div style="display: flex; flex-direction: row">
          <el-row>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div :id=gauges.cpu.id style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div :id=gauges.mem.id style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div :id=gauges.swap.id style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12" v-for="disk in gauges.disks">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div :id=disk.id style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div id="unusedChart1" style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div id="unusedChart2" style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
            <el-col :xl="4" :lg="6" :sm="8" :xs="12">
              <div style="margin: -36px -16px -36px -16px; text-align: center; width: 260px;">
                <div id="unusedChart3" style="width: 260px;height:260px;"></div>
              </div>
            </el-col>
          </el-row>
        </div>
        <div class="table">
          <el-row :gutter="20">
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">系统主机名</span>
                <span style="width: 60%">{{ host.hostName }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">操作系统</span>
                <span style="width: 60%">{{ host.platform }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">系统时间</span>
                <span style="width: 60%">{{ dateToFormat(serverTime) }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">内核</span>
                <span style="width: 60%">{{ host.os }}, {{ host.kernelArch }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">CPU 信息</span>
                <span style="width: 60%">{{ cpu.modelName }}, {{ cpu.cores }} 核心</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">系统在线时间</span>
                <span style="width: 60%">{{ timeToFormat(host.uptime) }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">正在运行的进程</span>
                <span style="width: 60%">{{ host.process }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">CPU 负载</span>
                <span style="width: 60%">{{ cpu.load1.toFixed(2) }} (1分钟) {{
                    cpu.load5.toFixed(2)
                  }} (5分钟) {{ cpu.load15.toFixed(2) }} (15分钟)</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">物理内存</span>
                <span style="width: 60%">已用 {{ byteToFormat(memory.used) }} / 空闲 {{
                    byteToFormat(memory.free)
                  }} / 总计 {{ byteToFormat(memory.total) }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">虚拟内存</span>
                <span style="width: 60%">已用 {{ byteToFormat(swap.used) }} / 空闲 {{
                    byteToFormat(swap.free)
                  }} / 总计 {{ byteToFormat(swap.total) }}</span>
              </div>
            </el-col>
            <el-col :span="24" :md="12" v-for="disk in disks">
              <div style="display: flex; flex-direction: row">
                <span style="width: 40%">本地磁盘 {{ disk.devices }} 空间</span>
                <span style="width: 60%">已用 {{ byteToFormat(disk.used) }} / 空闲 {{
                    byteToFormat(disk.free)
                  }} / 总计 {{ byteToFormat(disk.total) }}</span>
              </div>
            </el-col>
          </el-row>
        </div>
      </div>
    </div>

    <div class="box">
      <h3 class="panel-title">历史状态</h3>
      <el-divider></el-divider>
      <div style="display: flex; flex-direction: row; margin-top: -30px">
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
      cpuUsage: 0,
      memUsage: 0,
      swapUsage: 0,
      serverTime: 0,
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
      swap: {
        total: 0,
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
      disks: [{
        devices: "unknown",
        mountPoint: "unknown",
        fsType: "unknown",
        total: 0,
        used: 0,
        free: 0
      }],
      gauges: {
        cpu: {
          id: 'cpuChart'
        },
        mem: {
          id: 'memChart'
        },
        swap: {
          id: 'swapChart'
        },
        disks: [{
          id: ''
        }]
      }
    }
  },
  methods: {
    async updateMoreInfo() {
      const _this = this
      await axios.get('http://127.0.0.1:9527/v1/moreInfo').then(function (response) {
        _this.cpu.cores = response.data['cpu']['cores']
        _this.cpu.modelName = response.data['cpu']['model_name']
        _this.cpu.load1 = response.data['cpu']['load_1']
        _this.cpu.load5 = response.data['cpu']['load_5']
        _this.cpu.load15 = response.data['cpu']['load_15']
        _this.memory.total = response.data['Memory']['total']
        _this.memory.available = response.data['Memory']['available']
        _this.memory.used = response.data['Memory']['used']
        _this.memory.free = response.data['Memory']['free']
        _this.swap.total = response.data['swap']['total']
        _this.swap.used = response.data['swap']['used']
        _this.swap.free = response.data['swap']['free']
        _this.host.hostName = response.data['Host']['host_name']
        _this.host.uptime = response.data['Host']['uptime']
        _this.host.process = response.data['Host']['process']
        _this.host.os = response.data['Host']['os']
        _this.host.kernelVersion = response.data['Host']['kernel_version']
        _this.host.kernelArch = response.data['Host']['kernel_arch']
        _this.host.platform = response.data['Host']['platform']
        _this.host.platformVersion = response.data['Host']['platform_version']

        let disks = [];
        for (let i = 0; i < response.data['Disks'].length; i++) {
          disks.push({
            devices: response.data['Disks'][i]['devices'],
            mountPoint: response.data['Disks'][i]['mount_point'],
            fsType: response.data['Disks'][i]['fs_type'],
            total: response.data['Disks'][i]['total'],
            used: response.data['Disks'][i]['used'],
            free: response.data['Disks'][i]['free']
          })
        }
        _this.disks = disks
        _this.serverTime = response.data['server_time']
      })
    },
    updateDate() {
      const _this = this;
      let cpuChart = echarts.init(document.getElementById(this.gauges.cpu.id)!);
      let memChart = echarts.init(document.getElementById(this.gauges.mem.id)!);
      let swapChart = echarts.init(document.getElementById(this.gauges.swap.id)!);
      let cpuHistoryElem = document.getElementById('cpuHistory')!;
      let memHistoryElem = document.getElementById('memHistory')!;
      let cpuHistory = echarts.init(cpuHistoryElem);
      let memHistory = echarts.init(memHistoryElem);

      setInterval(function () {
        axios.get('http://127.0.0.1:9527/v1/current')
            .then(function (response) {
              // console.log(response.data);
              _this.cpuUsage = response.data['cpu_usage'];
              _this.memUsage = response.data['mem_usage'];
              _this.swapUsage = response.data['swap_usage'];
              cpuChart.setOption({
                series: [{
                  data: [{
                    value: _this.cpuUsage
                  }]
                }]
              })
              memChart.setOption({
                series: [{
                  data: [{
                    value: _this.memUsage
                  }]
                }]
              })
              swapChart.setOption({
                series: [{
                  data: [{
                    value: _this.swapUsage
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
                cpuUsageList[i] = [i, response.data[i]['cpu_usage']];
                memUsageList[i] = [i, response.data[i]['mem_usage']];
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
        _this.updateMoreInfo()
      }, 10000)
    },

    async initChart() {
      await this.updateMoreInfo();

      let cpuChart = echarts.init(document.getElementById(this.gauges.cpu.id)!);
      let memChart = echarts.init(document.getElementById(this.gauges.mem.id)!);
      let swapChart = echarts.init(document.getElementById(this.gauges.swap.id)!);
      let unusedChart1 = echarts.init(document.getElementById('unusedChart1')!);
      let unusedChart2 = echarts.init(document.getElementById('unusedChart2')!);
      let unusedChart3 = echarts.init(document.getElementById('unusedChart3')!);
      let cpuHistory = echarts.init(document.getElementById('cpuHistory')!);
      let memHistory = echarts.init(document.getElementById('memHistory')!);
      let gaugeOption = {
        series: [{
          type: 'gauge',
          itemStyle: {
            shadowColor: 'rgba(0,138,255,0.45)',
            shadowBlur: 5,
            shadowOffsetX: 1,
            shadowOffsetY: 1
          },
          progress: {
            show: true,
            roundCap: true,
            width: 8
          },
          pointer: {
            icon: 'path://M2090.36389,615.30999 L2090.36389,615.30999 C2091.48372,615.30999 2092.40383,616.194028 2092.44859,617.312956 L2096.90698,728.755929 C2097.05155,732.369577 2094.2393,735.416212 2090.62566,735.56078 C2090.53845,735.564269 2090.45117,735.566014 2090.36389,735.566014 L2090.36389,735.566014 C2086.74736,735.566014 2083.81557,732.63423 2083.81557,729.017692 C2083.81557,728.930412 2083.81732,728.84314 2083.82081,728.755929 L2088.2792,617.312956 C2088.32396,616.194028 2089.24407,615.30999 2090.36389,615.30999 Z',
            length: '80%',
            width: 8,
            offsetCenter: [0, '5%']
          },
          axisLine: {
            roundCap: true,
            lineStyle: {
              width: 8
            }
          },
          axisTick: {
            splitNumber: 2,
            length: 5,
            lineStyle: {
              width: .3,
              color: '#999'
            }
          },
          splitLine: {
            length: 7,
            lineStyle: {
              width: 1,
              color: '#999'
            }
          },
          axisLabel: {
            show: false
          },
          detail: {
            valueAnimation: true,
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}\n{name|凑数的}';
            },
            rich: {
              value: {
                fontFamily: 'Roboto',
                fontSize: 24,
                color: '#777',
                padding: [28, 0, 0, 16]
              },
              unit: {
                fontFamily: 'Roboto',
                fontSize: 16,
                color: '#999',
                padding: [28, 0, -4, 2]
              },
              name: {
                fontSize: 16,
                fontWeight: 500,
                color: '#777',
                padding: [40, 0, 0, 0]
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

      cpuChart.setOption(gaugeOption);
      cpuChart.setOption({
        series: [{
          itemStyle: {
            color: '#58D9F9',
            shadowColor: 'rgba(0,138,255,0.45)',
          },
          detail: {
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}\n{name|CPU}';
            },
          }
        }]
      });
      memChart.setOption(gaugeOption);
      memChart.setOption({
        series: [{
          itemStyle: {
            color: '#F97858',
            shadowColor: 'rgba(255,119,0,0.45)',
          },
          detail: {
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}\n{name|物理内存}';
            },
          }
        }]
      });
      swapChart.setOption(gaugeOption);
      swapChart.setOption({
        series: [{
          itemStyle: {
            color: '#7858F9',
            shadowColor: 'rgba(141,90,248,0.45)',
          },
          detail: {
            formatter: function (value: number) {
              return '{value|' + value.toFixed(0) + '}{unit|%}\n{name|虚拟内存}';
            },
          }
        }]
      });
      unusedChart1.setOption(gaugeOption);
      unusedChart2.setOption(gaugeOption);
      unusedChart3.setOption(gaugeOption);

      cpuHistory.setOption(option3);
      memHistory.setOption(option4);

      let diskGauges = [];
      for (let i = 0; i < this.disks.length; i++) {
        diskGauges.push({id: 'diskChart' + i})
      }
      this.gauges.disks = diskGauges
      for (let i = 0; i < this.gauges.disks.length; i++) {
        let diskChart = echarts.init(document.getElementById(this.gauges.disks[i].id)!);
        diskChart.setOption(gaugeOption);
      }
    },

    byteToFormat(bytes: number) {
      let unit = ['B', 'KiB', 'MiB', 'GiB', 'TiB', 'PiB'];
      let i = 0;
      while (bytes >= 1024) {
        bytes /= 1024.0;
        i++;
      }
      return bytes.toFixed(2) + ' ' + unit[i];
    },

    timeToFormat(time: number) {
      let days, hours, minutes, string;
      time = Math.floor(time / 60);
      minutes = time % 60;
      time = Math.floor(time / 60);
      hours = time % 24;
      time = Math.floor(time / 24);
      days = time;
      string = minutes + ' 分钟';
      if (hours > 0) {
        string = hours + ' 小时, ' + string;
      }
      if (days > 0) {
        string = days + ' 天, ' + string;
      }
      return string;
    },
    dateToFormat(timestamp: number) {
      let date = new Date(timestamp * 1000)
      return date.getFullYear() + '年' + date.getMonth() + '月' + date.getDay() + '日' +
          (date.getHours() >= 12 ? '下午' : '上午') + (date.getHours() >= 12 ? date.getHours() - 12 : date.getHours()) +
          '点' + date.getMinutes() + '分';
    }
  },
  mounted() {
    this.updateDate();
    this.initChart();
  }
})

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto&display=swap');

.box {
  padding: 10px;
  margin: 8px 10px 2px 10px;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.table {
  /*margin-top: -40px;*/
  margin-left: 5px;
  font-family: 'Roboto', sans-serif;
  font-size: 13px;
  font-weight: normal;
  line-height: 29px;
}

.panel-title {
  margin-top: 0;
  margin-bottom: -14px;
  font-size: 18px;
  color: inherit;
  font-weight: normal;
  font-family: inherit;
}
</style>
