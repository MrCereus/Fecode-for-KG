<template>
  <!-- 整个网页 -->
  <a-flex vertical class="whole">
    <!-- 上侧 -->
    <a-flex class="block" :style="{ flex: 1 }">
      <selectView class="block" :style="{ flex: 1 }" />
      <menuView class="block" :style="{ flex: 5 }" />
    </a-flex>
    <!-- 下侧 -->
    <a-flex class="block" :style="{ flex: 13 }">
      <!-- 表格 -->
      <KGTable class="block" :style="{ flex: 1 }" :data="tlbData" />
      <a-flex vertical class="block" :style="{ flex: 5 }">
        <!-- 详细视图 -->
        <detailView
          :WCDatas="WCDatas"
          :tlbData="tlbData.slice(0, 5)"
          :ENDatas="ENDatas"
          :RLDatas="RLDatas"
        />
        <!-- 力导向图 -->
        <a-flex class="block" :style="{ flex: 1 }">
          <FView
            class="block"
            :style="{ width: '50%' }"
            assignId="LFView"
            :FGData="FGDataL"
            type="base"
            :centerNodePairIds="['6661', '17161']"
            :alignNodePairListIds="[
              ['306', '10806'],
              ['773', '11273'],
              ['1634', '12134'],
              ['4411', '14911'],
              ['5819', '16319'],
              ['24113', '35194'],
            ]"
            @startFollow="startFollow"
          />
          <FView
            class="block"
            :style="{ width: '50%' }"
            assignId="RFView"
            type="follow"
            :FGData="FGDataR"
            :followNodes="followNodes"
          />
        </a-flex>
      </a-flex>
    </a-flex>
  </a-flex>
</template>

<script>
import { getFGData, getTlbData, getWCData, getLiData } from "./api/utils.js";
import detailView from "./components/detail/detailView.vue";
import FView from "./components/down/FView.vue";
import KGTable from "./components/side/KGTable.vue";
import selectView from "./components/top/selectView.vue";
import menuView from "./components/top/menuView.vue";
import { ref, onMounted } from "vue";

export default {
  setup() {
    // 读取Json数据
    const followNodes = ref({});
    const tlbData = ref([]);
    const WCDatas = ref([]);
    const FGDataL = ref({});
    const FGDataR = ref({});
    const ENDatas = ref([]);
    const RLDatas = ref([]);
    function startFollow(data) {
      followNodes.value = data;
    }
    onMounted(async () => {
      // 读取表格数据
      tlbData.value = await getTlbData();

      // 读取词云数据
      const WCList = [];
      for (let i = 1; i <= 5; ++i) {
        const res = await getWCData("WordCloud" + i + ".json");
        WCList.push(res);
      }
      WCDatas.value = WCList;

      // 读取力导向图数据
      FGDataL.value = await getFGData("ForceGraph1.json");
      FGDataR.value = await getFGData("ForceGraph2.json");

      // 读取节点列表数据
      const ENList = [];
      for (let i = 1; i <= 5; ++i) {
        const res = await getLiData("./EL/EntityList" + i + ".json");
        ENList.push(res);
      }
      ENDatas.value = ENList;

      // 读取边列表数据
      const RLList = [];
      for (let i = 1; i <= 5; ++i) {
        const res = await getLiData("./RL/RelList" + i + ".json");
        RLList.push(res);
      }
      RLDatas.value = RLList;
    });

    return {
      tlbData,
      WCDatas,
      FGDataL,
      FGDataR,
      ENDatas,
      RLDatas,
      startFollow,
      followNodes,
    };
  },
  components: {
    detailView,
    FView,
    KGTable,
    selectView,
    menuView,
  },
};
</script>

<style scoped>
/* 砖块，为了更加美观 */
.block {
  border: 1px solid #ccc;
}

/* 整个网页，设置这个属性为了占满空间 */
.whole {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0px;
  left: 0px;
}
</style>./api/utils.js
