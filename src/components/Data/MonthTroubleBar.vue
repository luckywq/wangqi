<template>
  <div class="month-trouble-box">

  </div>
</template>

<script>
  import {barCharts} from '@/public/charts';
  export default {
    data () {
      return {
        monthTroubleBar: null,
        months: ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '10月'],
        monthsData: [11, 22, 33, 44, 55, 66, 77, 88],
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
          if (this.monthTroubleBar !== null) {
            this.monthTroubleBar.resize();
          }
        },
        async getData () {
          const res = await this.$http('/villages', {
              data: { 'opts': this.queryCorrelation }
          });
          this.villages = res.result.rows;
          const monthTroubleBox = document.getElementsByClassName('month-trouble-box')[0];
          this.monthTroubleBar = barCharts(monthTroubleBox, this.months, this.monthsData, '#f3c21d','#f2f2f2');
        }
    },
    mounted () {
        this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
        this.getData();
    }
  };
</script>

<style>
    .month-trouble-box{
        width: 100%;
        height: 100%;
    }
</style>
