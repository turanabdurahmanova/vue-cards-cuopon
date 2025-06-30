<template lang="pug">
  .card-order-form-wrap
    AppSimaAlert(v-if="!userData.verificationStatus > 0")
    v-form.row.card-order-form-row(ref="order" v-model="orderForm" @submit.native.prevent="order" :class="!userData.verificationStatus > 0 ? 'borderRadiusReset' : ''")
        h2.card-title {{ $t('label.fill.information.order') }}
        v-text-field.field(
        outlined
        disabled
        :label="$t('label.name')"
        v-model="userData.firstname"
        required)
        v-text-field.field(
        outlined
        disabled
        :label="$t('label.surname')"
         v-model="userData.lastname"
        required)
        v-text-field.field(
        prefix="+994"
        outlined
        v-model="orderData.secondPhone"
        :rules="rules.secondPhone"
        v-maska:[options]="orderData.secondPhone"
        :label="$t('label.phone.number')"
        required
        :validate-on-blur="true"
        @input="message = ''; responseError=[]"
        :error-messages="responseError"
        :error="message!==''")
        v-text-field.field(
        outlined
        disabled
        :label="$t('label.pin.code')"
        v-model="userData.personalId"
        required)
        v-text-field.field(
        :label="$t('label.branch.delivery')"
        outlined
        required
        disabled
        v-model="orderData.branche")
        v-text-field.field(
        outlined
        v-model="orderData.keyWord"
        :label="$t('label.secret.word')"
        required
        :rules = "rules.keyWord")
        v-btn.card-order-form-btn(id="sign_up" depressed block x-large color="primary" type="submit" :disabled="!orderForm || !userData.verificationStatus > 0") {{$t('action.order.confirm')}}
        span.soon(v-if="!userData.verificationStatus > 0") {{$t('action.soon')}}
        component(
          :is="currentForm"
          :is-modal-visible.sync='isModalVisible')
</template>

<script>
import { mapGetters } from 'vuex';
import { ValidationRequired } from '@/rules/required';
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppSimaAlert from '@/components/AppSimaAlert';
import CardOrderSuccess from '@/views/dashboard/pages/cards/modal/card-order-success';
import { ValidationPattern } from '@/rules/pattern';

export default {
  name: 'CardOrderForm',
  components: {
    CardOrderSuccess,
    AppSimaAlert,
  },
  data() {
    return {
      orderForm: false,
      message: '',
      responseError: [],
      isModalVisible: false,
      currentForm: '',
      orderData: {
        keyWord: '',
        secondPhone: null,
        branche: 'Rabitəbank, KOB Mərkəzi filialı',
      },
      rules: {
        secondPhone: [ValidationRequired(), ValidationPattern('(?:\\s*\\d){9}', 'validation.login.number')],
        keyWord: [ValidationRequired()],
      },
      options: {
        eager: true,
        mask: '## ### ## ##',
      },
    };
  },
  computed: {
    ...mapGetters('user', {
      userData: 'getUserData',
    }),
  },
  methods: {
    async order() {
      if (this.userData.verificationStatus > 0) {
        if (!this.$refs.order.validate()) {
          return;
        }
        const req = {
          keyWord: this.orderData.keyWord,
          secondPhone: this.orderData.secondPhone,
        };
        console.log('cardOrder req', req);
        await MpayCobrandRequests.cardOrder(req).then((response) => {
          if (response?.data) {
            this.currentForm = 'CardOrderSuccess';
            this.isModalVisible = true;
            this.orderForm = true;
          }
        }).catch((error) => {
          console.log(error);
        }).finally(() => {
          this.$notify.success(this.$t('misc.notify.general.success'));
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/cards.scss';
:deep(.v-label.theme--light){
  color:#262626;
  font-size: 14px;
  font-weight: 500;
  line-height: 120%; /* 16.8px */
}
.v-input .v-label{
  font-size: 14px;
  top: -29px !important;
}
.card-title{
  margin-bottom: 32px;
}

:deep(.theme--light.v-input--is-disabled .v-input__slot){
  color: #BCBCBC;
  font-size: 14px;
  font-weight: 400;
  line-height: 125%;
  background:#FAFAFA;
}

.card-order-form-row .soon{
  content:"";
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0.5;
  background: #1F2024;
  z-index: 1;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  left: 0;
  bottom: 0;
  transform: scale(0);
  transition-duration: 0s;
  transition: .25s linear;
}
.card-order-form-row:hover .soon{
  // transition-duration: 700ms;
  transform: scale(1);
}
@media (max-width: 650px) {
  .v-text-field.v-text-field--enclosed.v-text-field--outlined.field{
      max-width: 100%;
  }
  :deep(.v-text-field__slot){
    height: 52px;
  }
  :deep(.v-input .v-label){
    display: flex;
    align-items: center;
    justify-content: center;
    height: 17px;
  }
  .card-title{
    margin-bottom: 24px !important;
  }
  .card-order-form-row.borderRadiusReset{
    border-radius: 0;
    box-shadow: none;
    padding: 32px 0;
  }
}
</style>
