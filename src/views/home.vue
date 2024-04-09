<template>
  <div class="dashboard-container">
    <div class="dashboard-editor-container">
      <panel-group @handleSetLineChartData="handleSetLineChartData"/>
      <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
        <line-chart :chart-data="lineChartData"/>
      </el-row>
    </div>
  </div>
</template>

<script>
import PanelGroup from './dashboard/PanelGroup'
import LineChart from './dashboard/LineChart'

const defaultForm = { id: null, username: null, nickName: null, gender: '男', email: null, enabled: 'false', roles: [], jobs: [], dept: { id: null }, phone: null }


const lineChartData = {
  user: {
    actualData: [120, 82, 91, 154, 162, 140, 145]
  },
  record: {
    actualData: [180, 160, 151, 106, 145, 150, 130]
  },
}

export default {
  name: 'Dashboard',
  components: {
    PanelGroup,
    LineChart,
  },
  data() {
    return {
      lineChartData: lineChartData.user
    }
  },
  methods: {
    handleSetLineChartData(type) {
      this.lineChartData = lineChartData[type]
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width: 1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
