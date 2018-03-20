<template>
  <!-- 污水处理量柱状图 -->
  <div class="sewage-treatment-box">

  </div>
</template>

<script>
  import {barChartsWithZoom} from '@/public/charts';
  export default{
    props: ['sewageTreatment'],
    data () {
      return {
        sewageTreatmenBar: null,
        villages: [],
        villagesName: [],
        villagesSewageTreatment: [],
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
        if(this.sewageTreatmenBar !== null) {
          this.sewageTreatmenBar.resize();
        }
      },
      async getData () {
          const res = await this.$http('/villages', {
            data: { 'opts': this.queryCorrelation }
          });
          this.villages = res.result.rows;
          for (let i = 0;i < this.villages.length;i ++) {
            this.villagesName.push(this.villages[i].name);
            this.villagesSewageTreatment.push(this.villages[i].area);
          }
          const sewageTreatmentBox = document.getElementsByClassName('sewage-treatment-box')[0];
          //this.sewageTreatmenBar = this.$parent.buildBarCharts(sewageTreatmentBox, this.villagesName,this.villagesSewageTreatment,'#f6c23b');
          this.sewageTreatmenBar = barChartsWithZoom(sewageTreatmentBox, this.villagesName,this.villagesSewageTreatment,'#f6c23b','#f2f2f2');
      
      }
    },
    mounted () {
      this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
      this.getData();
    }
  };
</script>

<style>
  .sewage-treatment-box{
    width: 100%;
    height: 230px;
  }
</style>
