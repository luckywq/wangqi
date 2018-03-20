<template>
<!-- 月维修分析柱状图 -->
  <div class="month-maintain-box">

  </div>
</template>

<script>
  import {barCharts} from '@/public/charts';
  export default {
    data () {
      return {
          monthMaintainBar: null,
          months: ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '10月'],
          monthsData: [11, 22, 33, 44, 55, 66, 77, 99],
          queryCorrelation: {
              where: {
                  pub_company_id: '',
                  available: true
              }
          },
          villages: []
      };
    },
    methods: {
        addResize () {
          if (this.monthMaintainBar !== null) {
              this.monthMaintainBar.resize();
          }
        },
        async getData () {
          const res = await this.$http('/villages', {
              data: { 'opts': this.queryCorrelation }
          });
          this.villages = res.result.rows;
          const monthMaintainBox = document.getElementsByClassName('month-maintain-box')[0];
          this.monthMaintainBar = barCharts(monthMaintainBox, this.months, this.monthsData, '#00d4f1','#f2f2f2');
        }
    },
    mounted () {
        this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
        this.getData();
    }
  };
</script>

<style>
    .month-maintain-box{
        width: 100%;
        height: 100%;
    }
</style>
