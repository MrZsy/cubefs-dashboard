<!--
 Copyright 2023 The CubeFS Authors.

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
 implied. See the License for the specific language governing
 permissions and limitations under the License.
-->

<template>
  <div>
    <el-card class="height mg-lf-s">
      <div class="justify-content-around">
        <CircleCharts
          v-for="(item, index) in chartData"
          :key="index"
          :e-data="item.data"
          :title="item.title"
          :color="['#00DAB3', '#ff0000']"
        ></CircleCharts>
      </div>
      <div class="justify-content-around" style="margin-top: 20px;">
        <CircleCharts
          v-for="(item, index) in chartData1"
          :key="index"
          :e-data="item.data"
          :title="item.title"
        >
        </CircleCharts>
      </div>
    </el-card>

    <el-dialog
      v-if="true"
      :title="$t('common.broken')"
      width="65%"
      top="10vh"
      center
    >
      <div style="margin-bottom: 10px; text-align: right;">
        <el-button type="primary" >{{ $t('common.batch') }}{{ $t('common.offline') }}</el-button>
      </div>
      <el-table
        :data="true"
        style="width: 100%"
      >
        <el-table-column type="selection" width="80"></el-table-column>
        <el-table-column
          :label="$t('common.id')"
          type="index"
        >
        </el-table-column>
        <el-table-column
          label="Path"
          prop="Path"
        ></el-table-column>
        <el-table-column label="IDs">
          <template slot-scope="scope">
            <span v-for="item in scope.row.PartitionIDs" :key="item">{{ item }}
            </span></template>
        </el-table-column>
      </el-table>
    </el-dialog>
    <el-dialog
      v-if="true"
      :title="$t('common.broken')"
      width="65%"
      top="10vh"
      center >
      <div style="margin-bottom: 10px; text-align: right;">
        <el-button type="primary" >{{ $t('common.batch') }}{{ $t('common.offline') }}</el-button>
      </div>
      <el-table
        :data="测试"
        style="width: 100%"
      >
        <el-table-column type="selection" width="80"></el-table-column>
        <el-table-column
          :label="$t('common.id')"
          type="index"
        >
        </el-table-column>
        <el-table-column
          label="Path"
          prop="Path"
        ></el-table-column>
        <el-table-column label="IDs">
          <template slot-scope="scope">
            <span v-for="item in scope.row.PartitionIDs" :key="item">{{ item }}
            </span></template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>
<script>
import {getBlobClusterStat, getBlobNodeList, getBlobService} from '@/api/cfs/cluster'
import { mapGetters } from 'vuex'
import CircleCharts from '@/components/circleCharts.vue'

export default {
  name: '',
  components: { CircleCharts },
  props: {
    clusterInfo: {
      type: Object,
      default() {
        return {}
      },
    },
  },
  data() {
    return {
      chartData: [],
      chartData1: [],
      mgrNode: 0,
      DataPartitionDialogVisible: false,
      DiskDialogVisible: false,
      getBlobDataNodeList: 0,
      blobNodeTotal: 0,
      health: this.$t('common.health'),
      unhealthy: this.$t('common.unhealthy'),
    }
  },
  computed: {
    ...mapGetters('clusterInfoModule', {
      curClusterInfo: 'clusterInfog',
    }),
  },
  watch: {},
  async created() {
    await this.getMgrPartition()
    await this.getBlobNodePartition()
    await this.getBlobPartition()
  },
  methods: {
    async getMgrPartition() {
      const res = await getBlobClusterStat({
        cluster_name: this.curClusterInfo.clusterName,
        cluster_id: this.clusterInfo.id,
      })
      const tempData = res.data.raft_status.peers
      const availableDisk = res.data.space_stat.disk_stat_infos[0].available
      const readonlyDisk = res.data.space_stat.disk_stat_infos[0].readonly
      let normalNode = 0
      let badNode = 0
      tempData.forEach(item => {
        if (item.active === true) {
          normalNode++
        } else {
          badNode++
        }
      })

      this.chartData.push({ data: [{ name: 'common.health', value: normalNode }, { name: 'common.unhealthy', value: badNode }], title: 'common.metamgrnodes' })
      this.chartData1.push({ data: [{ name: 'common.health', value: availableDisk }, { name: 'common.unhealthy', value: readonlyDisk }], title: 'common.blobstoredisks' })
    },
    async getBlobNodePartition() {
      const res = await getBlobNodeList({
        cluster_name: this.curClusterInfo.clusterName,
        cluster_id: this.clusterInfo.id,
      })
      const blobNode = res.data.total
      this.chartData.push({ data: [{ name: 'common.health', value: blobNode }, { name: 'common.unhealthy', value: 0 }], title: 'common.blobstorenode' })
    },
    async getBlobPartition() {
      const res = await getBlobService({
        cluster_name: this.curClusterInfo.clusterName,
        cluster_id: this.clusterInfo.id,
      })
      const tempData = res.data
      let proxyNum = 0
      let badProxyNum = 0
      let schedulerNum = 0
      let badSchedulerNum = 0
      tempData.forEach(item => {
        if (item.name === 'SCHEDULER') {
          schedulerNum++
        } else if (item.name === 'PROXY') {
          proxyNum++
        }
      })
      this.chartData.push({ data: [{ name: 'common.health', value: schedulerNum }, { name: 'common.unhealthy', value: badSchedulerNum }], title: 'common.blobstoreproxy' })
      this.chartData1.push({ data: [{ name: 'common.health', value: proxyNum }, { name: 'common.unhealthy', value: badProxyNum }], title: 'common.blobstorescheduler' })
    }
  },
}
</script>
<style lang='scss' scoped>
.height {
  height: 600px;
  width: 100%;
}
</style>
