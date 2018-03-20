<template>
<!-- 污水处理量占比饼图 -->
  <div class="sewage-scale-box">

  </div>
</template>

<script>
  import {pieCharts} from '@/public/charts';
  export default{
    data () {
      return {
        sewagePieCharts: null,
        villages: [],
        villagesSewageScale: [],
        queryCorrelation: {
          where: {
            pub_company_id: '',
            available: true
          }
        }
      };
    },
    methods: {
      addResize () {
        if (this.sewagePieCharts !== null) {
          this.sewagePieCharts.resize();
        }
      },
      async getData () {
        const res = await this.$http('/villages', {
            data: { 'opts': this.queryCorrelation }
        });
        this.villages = res.result.rows;
        const result = res.result.rows;
        for (let i = 0;i < result.length;i ++) {
            this.villagesSewageScale[i] = {};
            this.villagesSewageScale[i].name = result[i].name;
            this.villagesSewageScale[i].value = result[i].area;
        }
        const sewageSacleBox = document.getElementsByClassName('sewage-scale-box')[0];
        this.sewagePieCharts = pieCharts(sewageSacleBox,this.villagesSewageScale,'#fff',true ,'#00d4f1',['40%','60%'],['50%','50%']);
      }
    },
    mounted () {
      this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
      this.getData();
    }
  };
</script>

<style>
  .sewage-scale-box{
    width: 100%;
    height: 100%;
  }
</style>

