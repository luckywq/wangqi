<template>
    <div class="info-box">
        <h2 class="box-tit">{{choiceVillage.name}}</h2>
        <ul class="water-flow">
            <li>出水流量：{{choiceVillage.value ? choiceVillage.value : "-"}}th</li>
            <li>进水流量：{{choiceVillage.value ? choiceVillage.value : "-"}}th</li>
        </ul>
        <ul class="dev-state" v-if="choiceVillage.status === 2">
            <li class="notice-txt" v-for='data in dataInfo' :key="data.id">
                <b class="iconfont icon-gaojing gaojing"></b>
                <span>{{data.pub_data_name}}</span>
            </li>
        </ul>
    </div>
</template>

<script>
    
    export default {
        props: ['choiceVillage'],
        data () {
            return {
                dataInfo: []
            };
        },
        methods: {
            async getData () {
                const res = await this.$http('/datasInfo',{
                    data: {opts: {where: {village_id: this.choiceVillage.id,status: 2}}}
                });
                this.dataInfo = res.result.rows;
            }
        },
        mounted () {
            this.getData();
        }
    };
</script>

<style>
.info-box{
  background: rgba(0,0,0,.8);
  width: auto;
  min-width: 137px;
  height: auto;
  border: 1px solid #005b7e;
  position: relative;
  color: #fff;
  text-align: center;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
}

.info-box:before{
  content: '';
  display: block;
  width: 8px;
  height: 8px;
  position: absolute;
  left:50%;
  margin-left: -5px;
  border: 1px solid #005b7e;
  bottom: -5px;
  transform: rotate(45deg);
  background: rgba(0,0,0,.8);
  border-left: 0;
  border-top: 0;
}

.info-box > h2{
  height: 26px;
  line-height: 26px;
  font-size: 15px;
  font-weight: normal;
    width:90%;
    margin: 0 auto; 
  border-bottom: 1px solid #005b7e;
}

.info-box > ul.water-flow{
  margin-top: 5px;
}

.info-box > ul.water-flow > li{
  font-size: 13px;
  line-height: 24px;
}

.info-box > ul.dev-state{
    margin-top: 2px;
    text-align: center;
    width: 100%;
    height: auto;
}

.marker-text{
  font-size: 15px !important;
}

.notice-txt{
  line-height: 30px;
  height: 30px;
  text-align: center;
  overflow: hidden;
}

.notice-txt > .gaojing{
  color:#ffd210;
  font-size: 15px;
  margin-right: 5px;
}
.notice-txt > span{
  color:#fff;
  font-size: 13px;
}

@media screen and (min-width : 1200px) and (max-width: 1500px) {
    .info-box > h2{
        height: 20px;
        line-height: 20px;
        font-size: 13px;
    }
    .info-box > ul.water-flow{
        margin-top: 2px;
    }
    .info-box > ul.water-flow > li{
        font-size: 12px;
        line-height: 18px;
    }
    .notice-txt{
        line-height: 22px;
        height: 22px;
    }
    .notice-txt > span{
        font-size: 12px;
    }
}
</style>
