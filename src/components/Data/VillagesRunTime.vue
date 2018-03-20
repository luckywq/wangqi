<template>
<!-- 各村设备运行时长折线图 -->
  <div class="run-time-box">

  </div>
</template>

<script>
import {lineChartsWithZoom} from '../../public/charts';
  export default {
      data () {
          return {
                villages: [],
                runTime: [],
                runTimeLineCharts: null,
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
            if (this.runTimeLineCharts !== null) {
                this.runTimeLineCharts.resize();
            }
        },
        async getData () {
            const res = await this.$http('/villages', {
                data: { 'opts': this.queryCorrelation }
            });
            const result = res.result.rows;
                for (let i = 0; i < result.length; i++) {
                    this.villages.push(result[i].name);
                    this.runTime.push(result[i].area);
                }
            const runTimeBox = document.getElementsByClassName('run-time-box')[0];
            this.runTimeLineCharts = lineChartsWithZoom(runTimeBox,this.villages,this.runTime);
        }
      },
      mounted () {
        this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
        this.getData();
      }
  };
</script>


<style>
    .run-time-box{
        width: 100%;
        height: 230px;
    }
</style>

