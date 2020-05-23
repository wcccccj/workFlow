<template>
  <el-dialog el-dialog title="提示" :visible.sync="tipVisible">
    <div class="ant-confirm-body">
        <i class="anticon anticon-close-circle" style="color: #f00;"></i>
        <span class="ant-confirm-title">当前无法发布</span>
        <div class="ant-confirm-content">
            <div>
                <p class="error-modal-desc">以下内容不完善，需进行修改</p>
                <div class="error-modal-list">
                    <div class="error-modal-item" v-for="(item,index) in tipList" :key="index">
                        <div class="error-modal-item-label">流程设计</div>
                        <div class="error-modal-item-content">{{item.name}} 未选择{{item.type}}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <span slot="footer" class="dialog-footer">
        <el-button @click="dialogClose">我知道了</el-button>
        <el-button type="primary" @click="dialogClose">前往修改</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  name: "component_name",
  props: {
    tipList: Array,
    visible: Boolean
  },
  watch: {
    'visible': function (val, oldVal) {
      this.tipVisible = val
    },
    'tipVisible': function (val, oldVal) {
      if (!val) {
        this.dialogClose()
      }
    }
  },
  data () {
    return {
      tipVisible: false
    };
  },
  methods: {
    dialogClose() {
      this.$emit('update:visible', false)
    }
  },
}
</script>
<style lang="scss" scoped>
</style>