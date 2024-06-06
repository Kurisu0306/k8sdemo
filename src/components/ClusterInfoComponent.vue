<script setup>
import * as echarts from 'echarts';
import { onMounted } from 'vue';
import axios from 'axios';

const k8sData = {
  pods: {},
  nodes: {},
  deployments: {},
  services: {}
};

const fetchData = async () => {
  try {
    const res = await axios.get('/cluster-info');
    const data = res.data;

    const countStatus = (items) => {
      const result = {};
      items.forEach(item => {
        if (result[item.status]) {
          result[item.status] += 1;
        } else {
          result[item.status] = 1;
        }
      });
      return result;
    };

    k8sData.pods = countStatus(data.pods.items);
    k8sData.nodes = countStatus(data.nodes.items);
    k8sData.services = countStatus(data.services.items);
    k8sData.deployments = countStatus(data.deployments.items);

    createChart('pods-chart', 'Pods', k8sData.pods);
    createChart('nodes-chart', 'Nodes', k8sData.nodes);
    createChart('deployments-chart', 'Deployments', k8sData.deployments);
    createChart('services-chart', 'Services', k8sData.services);
  } catch (err) {
    console.error(err);
  }
};

const createChart = (id, title, data) => {
  const chartDom = document.getElementById(id);
  if (!chartDom) return;
  const myChart = echarts.init(chartDom);
  const colors = {
    Running: 'green',
    Pending: 'yellow',
    Failed: 'red',
    Available: 'green',
    Unavailable: 'red',
    True: 'green',
    False: 'red',
    ClusterIP: 'green',
    NodePort: 'yellow',
    LoadBalancer: 'red'
  };

  const option = {
    title: {
      text: title,
      left: 'center'
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      orient: 'vertical',
      bottom: 'bottom'
    },
    series: [
      {
        name: title,
        type: 'pie',
        radius: ['50%', '70%'],
        avoidLabelOverlap: false,
        label: {
          show: false,
          position: 'center'
        },
        emphasis: {
          label: {
            show: true,
            fontSize: '16',
            fontWeight: 'bold'
          }
        },
        labelLine: {
          show: false
        },
        data: Object.entries(data).map(([key, value]) => ({ name: key, value, itemStyle: { color: colors[key] || 'grey' }
        }))
      }
    ]
  };
  myChart.setOption(option);
};

onMounted(() => {
  fetchData();
});
</script>

<template>
  <div class="container">
    <div id="pods-chart" class="chart"></div>
    <div id="nodes-chart" class="chart"></div>
    <div id="deployments-chart" class="chart"></div>
    <div id="services-chart" class="chart"></div>
  </div>
</template>

<style scoped>
.container {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
  width: 100vw;
  height: 90vh;
  justify-items: center;
  align-items: center;
}

.chart {
  width: 350px;
  height: 350px;
}
</style>
