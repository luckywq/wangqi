<template>
<!-- 设备可用率折线图 -->
  <div class="available-box" style="height:230px">

  </div>
</template>

<script>
  import {lineChartsWithZoomBG} from '@/public/charts';
  export default{
    data () {
      return {
        villagesName: [],
        availableScale: [],
        availableLine: null,
        queryCorrelation: {
          where: {
            pub_company_id: '',
            available: true
          }
        },
      };
    },
    methods: {
      addResize() {
        if (this.availableLine !== null) {
          this.availableLine.resize();
        }
      },
      async getData () {
        const res = await this.$http('/villages', {
              data: { 'opts': this.queryCorrelation }
        });
        for (let data of res.result.rows) {
          this.villagesName.push(data.name);
          this.availableScale.push(data.area);
        }
        const availableBox = document.getElementsByClassName('available-box')[0];
          this.availableLine = lineChartsWithZoomBG(availableBox, this.villagesName, this.availableScale);
      }
    },
    mounted () {
      this.queryCorrelation.where.pub_company_id = JSON.parse(sessionStorage.getItem('userInfo')).company_id;
      this.getData();
    }
  };
</script>

<style>
  .available-box{
    width: 100%;
    height:230px !important;
  }
</style>
