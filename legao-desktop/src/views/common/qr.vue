<template>
  <div class="legao-dialog qr-manage">
    <!-- 支付扫码 -->
    <common-dialog class="qr-dialog el-dialog-white" @close-self="QrClose"  :visible="qrVisible" :modal="false">
      <div class="main" slot="main">
        <p class="title"><label>收款金额:</label>&yen;<span>{{payData.money}}</span>元</p>
        <div class="qr-code">
          <label>付款条码:</label>
          <el-input
            ref="siteInput"
            clearable
            size="mini"
            placeholder="【请扫用户付款条码】"
            v-model="qrCodeValue">
          </el-input>
        </div>
        <div class="qr-img">
          <img src="../../images/member/code.png"/>
          <p>请扫描用户手机客户端中的<span style="color:red;">{{getPlat(payData.plat)}}</span>条码</p>
        </div>
      </div>
      <template slot="footer">
        <el-button @click="submitPay" :loading="loading" type="primary" >{{buttonText}}</el-button>
      </template>
    </common-dialog>
  </div>
</template>

<script>
import CommonDialog from "@/views/common/dialog";
import { mapGetters, mapActions } from "vuex";
import { fetchPayment } from "@/api/pay";
export default {
  components: {
    CommonDialog
  },
  computed: {
    ...mapGetters(["qrVisible", "payData"]),
    buttonText() {
      if (this.loading) {
        return "支付中...";
      }
      return this.qrCodeValue ? "确定支付" : "扫描中，请稍等...";
    }
  },
  watch: {
    qrVisible() {
      if (this.qrVisible) {
        this.$nextTick(function() {
          this.$refs["siteInput"].focus();
        });
      }
    }
  },
  data() {
    return {
      loading: false, //支付等待
      qrCodeValue: "" //条码
    };
  },
  methods: {
    ...mapActions(["QrClose"]),
    getPlat(type) {
      switch (type) {
        case "wx":
          return "微信";
        case "ali":
          return "支付宝";
      }
    },
    submitPay() {
      if (this.qrCodeValue && this.payData.card_no && this.payData.money) {
        this.loading = true;
        fetchPayment({
          card_no: this.payData.card_no,
          type: this.payData.plat,
          price: this.payData.money,
          authcode: this.qrCodeValue
        }).then(
          () => {
            this.$message({
              message: "支付成功!",
              type: "success",
              duration: 1000
            });
            setTimeout(() => {
              2;
              this.QrClose();
            }, 1000);
          },
          () => {
            this.$message({
              message: "支付失败!",
              type: "error",
              duration: 2000
            });
            this.loading = false;
          }
        );
      }
    }
  }
};
</script>
<style lang="scss" >
.qr-manage {
  .qr-dialog {
    .el-dialog {
      height: 5.7rem;
      width: 6.41rem;
    }
    .el-dialog__body {
      padding: 0;
      .title {
        label {
          margin-right: 0.1rem;
        }
        font-size: 0.24rem;
        span {
          color: red;
        }
      }
    }
    .qr-code {
      width: 60%;
      margin: 0 auto;
      margin-top: 0.3rem;
      @include flexCenter;
      label {
        display: inline-block;
        width: 1.5rem;
        font-size: 0.18rem;
      }
    }
    .qr-img {
      width: 80%;
      margin: 0 auto;
      padding-bottom: 0.5rem;
      border-bottom: 1px solid #c0c4cc;
      margin-top: 0.4rem;
      @include flexCenter(column);
      img {
        width: 3.57rem;
        height: 1.83rem;
      }
    }
    .el-dialog__footer {
      .dialog-footer {
        .el-button {
          width: 2.6rem;
        }
      }
    }
  }
}
</style>
