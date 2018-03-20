<template>
  <div class="infoBox">

    <!-----------------------------组态选择小窗口-------------------------------->
      <el-popover ref="popover4" placement="bottom">
          <div class="popver-scadas" v-if="scadas.length>0">
              <div  v-for="scada in scadas" :key="scada.id">
                <div v-if="scada.type===0" @click="showScada('CheckVillageScada',0)">村庄组态</div>
                <div v-else-if="scada.type===1" @click="showScada('CheckWaterScada',1)">供水设备组态</div>
                <div v-else-if="scada.type===2" @click="showScada('CheckSewageScada',2)">污水处理设备组态</div>
              </div>
          </div>
          <div v-else>
                该村庄暂未配置组态
          </div>
      </el-popover>

        <div v-if="chioseVillage.status===0" class="infoBox-title grayTitle">
          {{chioseVillage.name}}
        </div>
        <div v-if="chioseVillage.status===1" class="infoBox-title greenTitle">
          {{chioseVillage.name}}
        </div>
        <div v-if="chioseVillage.status===2" class="infoBox-title redTitle">
          {{chioseVillage.name}}
        </div>
        <div class="infoBox-content">
          <div class="content-main">
            <label>出水流量:{{
              chioseVillage.value ? chioseVillage.value : "-"
            }} t/h</label>
            <label>进水流量:{{
              chioseVillage.value ? chioseVillage.value : "-"
            }} t/h</label>
          </div>
          <div class="content-faults">
              <div class="faults-info" v-for="fault in faultData" :key="fault.id">
                  <img src="../../assets/image/sy_marker_notice.png"/>
                  <label v-text="fault.pub_data_name"></label>
              </div>
          </div>
          <div class="content-tools">
          <el-tooltip class="item" effect="dark" content="村庄信息" placement="top">
            <span class="fa fa-book" @click="openDialog('infoTab')"></span>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="维修记录" placement="top">
            <span class="fa fa-wrench" @click="openDialog('maintain')"></span>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="设备监控" placement="top">
            <span class="fa fa-desktop" v-popover:popover4 @click="getScada"></span>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="污泥外送" placement="top">
            <span class="fa fa-truck" @click="openDialog('sludge')"></span>
          </el-tooltip>
          </div>
        </div>
        
      </div>
</template>
<script>
export default {
  props: ['chioseVillage', 'faultData'],
  data() {
    return {
      scadas: []
    };
  },
  methods: {
    openDialog(val) {
      this.$emit('showDialog', val);
    },
    async getScada() {
      const res = await this.$http('/scadas', { data: { opts: { where: { village_id: this.chioseVillage.id } } } });
      this.scadas = res.result.rows;
    },
    showScada(comp,type) { 
      sessionStorage.setItem('village_name',this.chioseVillage.name);  
      sessionStorage.setItem('scada_type',type);
      this.$router.push({ name: comp, params: { villageId: this.chioseVillage.id } });  
    }
  },
};
</script>
<style>
@import "MapInfoWindow.css";
</style>
