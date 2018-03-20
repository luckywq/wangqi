<template>
    <div id="check_scada">
        <div class="pos-title">
            <breadcrumb :title="title"></breadcrumb>
        </div>
        <el-dialog :visible.sync='dialogAIVisible'
                    custom-class="scada-dialog"
                    width="800px"
                    :center="true"
                    :title='dialog_tit'>
            <ai-line-chart>
            </ai-line-chart>
        </el-dialog>
        <el-dialog :visible.sync='dialogDIVisible'
                    custom-class="scada-dialog"
                    width="800px"
                    :center="true"
                    :title='dialog_tit'>
            <di-bar-chart>
            </di-bar-chart>
        </el-dialog>   
        <div class="scada-wrapper">
            <iframe src="" frameborder="0" id="scada-check"></iframe>
        </div>
        
    </div>
</template>

<script>
import AiLineChart from './AiLineChart';
import DiBarChart from './DiBarChart';
export default {
    data() {
        return {
            dialogDIVisible: false,
            dialogAIVisible: false,
            scada: [],
            title: ['返回', ''],
            allData: {
                lanyueSrc: 'http://121.42.253.149:18822/scada',
                ajaxComplated: {},
                parentData: {
                    type: 3,
                    scada_id: '',
                    scada_config: {
                        data_list: [],
                        anchor_list: []
                    }
                },

            },
            village_id: 0,
            village_name: '',
            dialog_tit: '',
            queryCorrelation: {
                where: {
                    village_id: 0,
                    type: 0
                }
            }
        };
    },
    components: {
        AiLineChart,
        DiBarChart,
        Breadcrumb: () => import('../Home/Breadcrumb'),
    },
    methods: {
        async getVillageScada() {
            const res = await this.$http('/scadas', {
                data: { 'opts': this.queryCorrelation }
            });
            this.scada = res.result.rows;
        },
        init() {
            this.getData();
            this.searchAnchor();
        },
        async getData() {
            this.allData.ajaxComplated.data = false;
            const res = await this.$http('/datasInfo', {
                data: { 'opts': { where: { village_id: this.village_id } } }
            });
            this.createParentMessage(res.result.rows);
            this.getMessageFromChild();
        },
        createParentMessage(data) {
            if (data && data.length > 0) {
                for (let value of data) {
                    let datas = {};
                    if (value.data_unit == 'm³/h') {
                        value.data_unit = 'm3/h';
                    }
                    datas.label_id = value.pub_data_id;
                    datas.label_name = value.pub_data_name;
                    datas.label_value = value.value;
                    datas.label_unit = value.pub_data_unit;
                    datas.status = value.status;
                    datas.port_type = value.pub_port_type;
                    datas.high_battery = value.pub_high_battery;
                    datas.low_battery = value.pub_low_battery;
                    this.allData.parentData.scada_config.data_list.push(datas);
                }
            }
            this.allData.ajaxComplated.data = true;
            this.postMessageToChild();
        },
        //查询锚点
        async searchAnchor() {
            this.allData.ajaxComplated.anchor = false;
            const res = await this.$http('/scadas', {
                data: { 'opts': { where: { village_id: this.village_id } } }
            });
            this.createParentAnchor(res.result.rows);
        },
        createParentAnchor(data) {
            if (data && data.length > 0) {
                for(let i = 1;i < data.length;i++){
                    if (data.type === 0) { continue; }
                    let datas = {};
                    datas.label_id = data[i].pub_scada_id;
                    datas.label_name = data[i].type === 1 ? '供水设备' : '污水处理设备';
                    this.allData.parentData.scada_config.anchor_list.push(datas); 
                }
            }
            this.allData.ajaxComplated.anchor = true;
            this.postMessageToChild();
        },
        //子层点击标签.锚点父层接受消息
        getMessageFromChild() {
            window.onmessage = ev => {
                const code = ev.data.code;
                const data = ev.data.data;
                if (code === 300) {
                    const group = data.group;
                    const originData = data.originData;
                    const dataType = originData.port_type;
                    if (group === 'data_list') {
                        switch (dataType) {
                            case 'AI':
                                this.dialogAIVisible = true;
                                break;
                            case 'DI':
                                this.dialogDIVisible = true;
                                break;
                            default:
                                break;
                        }
                    }
                    if (group === 'anchor_list' && this.queryCorrelation.where.type === 0) {
                        //const village_id = this.village_id;
                        const village_name = this.village_name;
                        switch (originData.label_name) {
                            case '供水设备':
                                this.$confirm(`是否跳转到${village_name}村庄供水设备组态?`,'提示',{
                                    confirmButtonText: '确定',
                                    cancelButtonText: '取消',
                                    type: 'warning'
                                }).then(()=>{
                                    sessionStorage.setItem('village_name',village_name);  
                                    sessionStorage.setItem('scada_type',1);
                                    this.queryCorrelation.where.village_id = this.village_id;
                                    this.queryCorrelation.where.type = 1;
                                    // this.getVillageScada();
                                    // this.init();
                                    this.$router.replace({ name: 'CheckWaterScada', params: { villageId: this.village_id } });
                                }).catch(()=>{

                                });
                                break;
                            case '污水处理设备':
                                this.$confirm(`是否跳转到${village_name}村庄污水处理设备组态?`,'提示',{
                                    confirmButtonText: '确定',
                                    cancelButtonText: '取消',
                                    type: 'warning'
                                }).then(()=>{
                                    sessionStorage.setItem('village_name',village_name);  
                                    sessionStorage.setItem('scada_type',2);
                                    this.queryCorrelation.where.village_id = this.village_id;
                                    this.queryCorrelation.where.type = 2;
                                    this.$router.replace({ name: 'CheckSewageScada', params: { villageId: this.village_id } });
                                }).catch(()=>{
                                    
                                });
                                break;
                            default:
                                break;
                        }
                    }
                }
            };
        },
        canPostMessageToChild() {
            let res = true;
            for (let key in this.allData.ajaxComplated) {
                if (!this.allData.ajaxComplated[key]) {
                    res = false;
                }
            }
            return res;
        },
        //向子层传递消息
        postMessageToChild() {
            if (!this.canPostMessageToChild()) { return; }
            const allData = this.allData;
            const scada = document.getElementById('scada-check');
            scada.setAttribute('src', allData.lanyueSrc);
            scada.onload = () => {
                if (this.scada && this.scada.length === 1) {
                    allData.parentData.scada_id = this.scada[0].pub_scada_id;
                    scada.contentWindow.postMessage(allData.parentData, '*');
                }
            };
        },
        //设置弹窗标题
        setTitText() {
            switch (this.queryCorrelation.where.type) {
                case 0:
                    this.dialog_tit = this.village_name + '村庄组态';
                    this.title.splice(1, 1,  this.dialog_tit);  //面包屑显示标题
                    break;
                case 1:
                    this.dialog_tit = this.village_name + '供水设备组态';
                    this.title.splice(1, 1,  this.dialog_tit);
                    break;
                case 2:
                    this.dialog_tit = this.village_name + '污水处理设备组态';
                    this.title.splice(1, 1,  this.dialog_tit);
                    break;
                default:
                    break;
            }
        }

    },
    mounted() {
        const url = window.location.href;
        const parm = url.split('/');
        const village_id = Number(parm[parm.length - 1]);
        this.village_id = village_id;
        this.village_name = sessionStorage.getItem('village_name');
        this.queryCorrelation.where.village_id = village_id;
        this.queryCorrelation.where.type = Number(sessionStorage.getItem('scada_type'));
        //设置弹窗标题
        this.setTitText();

        this.getVillageScada();
        this.init();
    }
};
</script>

<style>
#check_scada {
  width: 100%;
  height: calc(100% - 80px);
  background-color: #ffffff;
  overflow: hidden;
}
#check_scada .pos-title {
  width: 95%;
  height: 45px;
  margin: auto;
  font-size: 16px;
  line-height: 70px;
  padding-top:10px;
  color: #000;
}
#check_scada .scada-wrapper {
  height: calc(100% - 50px);
  width: 98%;
  margin: auto;
  border-radius: 15px;
}
#check_scada .scada-wrapper > iframe {
  width: 100%;
  height: 100%;
  overflow: hidden;
  margin: 0;
  padding: 0;
}
#check_scada .scada-dialog {
  width: 800px;
  height: 500px !important;
  background: rgba(255, 255, 255, 1);
}

#check_scada .el-dialog__body {
  position: relative;
}
#check_scada .el-dialog__body div {
  width: 800px;
  height: 460px;
  position: absolute;
  left: 0;
  top: 0;
}
</style>
