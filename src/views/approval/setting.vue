<template>
    <div>
        <div style="margin: 50px;">
            <div class="fd-nav">
                <!-- <div class="fd-nav-left">
                    <div class="fd-nav-back" @click="toReturn"><i class="anticon anticon-left"></i></div>
                    <div class="fd-nav-title">{{workFlowDef.name}}</div>
                </div> -->
                <div class="fd-nav-center">
                </div>
                <div class="fd-nav-right">
                    <button type="button" class="ant-btn button-publish" @click="saveSet"><span>发 布</span></button>
                </div>
            </div>
            
            <div class="fd-nav-content">
                <div class="dingflow-design">
                    <!-- 流程盒子 -->
                    <div class="box-scale" id="box-scale" :style="'transform: scale('+nowVal/100+'); transform-origin: 50% 0px 0px;'">
                        <node-wrap :nodeConfig.sync="nodeConfig" :flowPermission.sync="flowPermission" 
                        :isTried.sync="isTried" :directorMaxLevel="directorMaxLevel" :tableId="tableId"></node-wrap>
                        <!-- 结束节点 -->
                        <div class="end-node">
                            <div class="end-node-circle"></div>
                            <div class="end-node-text">流程结束</div>
                        </div>
                    </div>
                </div>
            </div>
            <tip-dialog :visible.sync="tipVisible" :tipList="tipList"></tip-dialog>
        </div>
        <div class="scale-item">
            <!-- 缩放器 -->
            <div class="zoom">
                <div :class="'zoom-out'+ (nowVal==50?' disabled':'')" @click="zoomSize(1)"></div>
                <span>{{nowVal}}%</span>
                <div :class="'zoom-in'+ (nowVal==300?' disabled':'')" @click="zoomSize(2)"></div>
            </div>
        </div>
    </div>
</template>
<script>
import tipDialog from '@/components/tipDialog'
import nodeWrap from '@/components/nodeWrap'
export default {
    components: {
        nodeWrap,
        tipDialog
    },
    data() {
        return {
            isTried: false,
            tipList: [],
            tipVisible: false,
            nowVal: 100,
            processConfig: {},
            nodeConfig: {},
            workFlowDef: {},
            flowPermission: [],
            directorMaxLevel: 0,
            dialogVisible: true,
            tableId: ""
        };
    },
    created() {
        this.$axios.get("/data.json", {
            workFlowDefId: this.$route.params.workFlowDefId
        }).then(res => {
            this.processConfig = res.data;
            this.nodeConfig = this.processConfig.nodeConfig;
            this.flowPermission = this.processConfig.flowPermission;
            this.directorMaxLevel = this.processConfig.directorMaxLevel;
            this.workFlowDef = this.processConfig.workFlowDef
            this.tableId = this.processConfig.tableId
        })
    },
    methods: {
        toReturn() {
            //window.location.href = ""
        },
        reErr(data) {
            if (data.childNode) {
                if (data.childNode.type == 1) {//审批人
                    if (data.childNode.error) {
                        this.tipList.push({ name: data.childNode.nodeName, type: "审核人" })
                    }
                    this.reErr(data.childNode)
                } else if (data.childNode.type == 2) { //抄送人
                    if (data.childNode.error) {
                        this.tipList.push({ name: data.childNode.nodeName, type: "抄送人" })
                    }
                    this.reErr(data.childNode)
                } else if (data.childNode.type == 3) { //条件
                    this.reErr(data.childNode.childNode)
                } else if (data.childNode.type == 4) {
                    this.reErr(data.childNode)
                    for (var i = 0; i < data.childNode.conditionNodes.length; i++) {
                        if (data.childNode.conditionNodes[i].error) {
                            this.tipList.push({ name: data.childNode.conditionNodes[i].nodeName, type: "条件" })
                        }
                        this.reErr(data.childNode.conditionNodes[i])
                    }
                }
            } else {
                data.childNode = null
            }
        },
        saveSet() {
            this.isTried = true;
            this.tipList = [];
            this.reErr(this.nodeConfig);
            if (this.tipList.length != 0) {
                this.tipVisible = true;
                return;
            }
            this.processConfig.flowPermission = this.flowPermission
            console.log(JSON.stringify(this.processConfig))
            // this.$axios.post("", this.processConfig).then(res => {
            //     if (res.code == 200) {
            //         this.$message.success("设置成功");
            //         setTimeout(function () {
            //             window.location.href = ""
            //         }, 200)
            //     }
            // })
        },
        zoomSize(type) {
            if (type == 1) {
                if (this.nowVal <= 50) {
                    return;
                }
                this.nowVal -= 10;
            } else {
                if (this.nowVal >= 300) {
                    return;
                }
                this.nowVal += 10;
            }
        }
    }
};
</script>
<style scoped>
@import "~@/css/workflow.css";
@import '~@/css/override-element-ui.css';
.error-modal-list {
    width: 455px;
}
</style>