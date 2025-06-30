<template lang="pug">
div.receipt.coupon
  AppBreadCrumbs(
    pageName="label.coupons"
    :crumbs="items"
  )
  div(class="receipt-wrapper" v-if="coupon")
    .receipt-wrapper__heading
      v-icon {{getCouponStatus.icon}}
      h5.receipt-wrapper__title {{ getCouponStatus.title  }}
      h2.receipt-wrapper__amount {{getAmount}}
    v-list
      template(v-for="(item, index) in getReceipt")
        v-list-item
          span.v-list-item__text {{item[0]}}
          template(v-if="index !== getReceipt.length - 1")
            span {{item[1]}}
              v-icon(v-if="index === 0" @click="copyReceiptCode(item[1])").receipt-id-copy.ml-2 $copy
          .v-list-item__logo(v-else)
            v-icon {{item[1].logo}}
            span {{item[1].content}}
    .receipt-wrapper__footer
      .receipt-wrapper__footer__item(v-if="getTopupByStatus && !isCouponTopUpCompleted")
        v-progress-circular(:width="3" :size="30" color="primary" v-if="isCouponTopUpLoading" indeterminate)
        template(v-else)
          v-icon(@click="topUpCoupon") $coupon-wallet
          span.btn-title {{$t('action.transfer.wallet')}}
      .receipt-wrapper__footer__item
        v-progress-circular(:width="3" :size="30" color="primary" v-if="isLoading" indeterminate)
        template(v-else)
          img(src="@/assets/download.svg" @click="downloadReceipt")
          span.btn-title {{$t('action.download')}}
</template>

<script>
import AppBreadCrumbs from '@/components/AppBreadCrumbs';
import { CouponRequests } from '@/services/http/coupon-requests';
import AppReceipt from '@/views/dashboard/components/AppReceipt';
import moment from 'moment';
import { PaymentRequests } from '@/services/http/payment-requests';
import config from '@/config/variables';
// import AppGoBackIcon from '@/components/AppGoBackIcon';

export default {
  name: 'CouponReceiptPage',
  components: {
    AppReceipt,
    // AppGoBackIcon,
    AppBreadCrumbs,
  },
  data() {
    return {
      isLoading: false,
      couponId: this.$route.params.id,
      coupon: null,
      email: '',
      errorCases: config.statusCases.error,
      pendingCases: config.statusCases.pending,
      isCouponTopUpCompleted: false,
      isCouponTopUpLoading: false,
      items: [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          href: '',
        },
        {
          text: this.$t('label.transfers'),
          disabled: true,
          href: '',
        },
      ],
    };
  },
  computed: {
    getTopupByStatus() {
      if (this.coupon.sender) {
        return this.coupon.status === 'REJECTED';
      }
      return this.coupon.status === 'READY';
    },
    getAmount() {
      if (this.coupon) {
        let value = this.coupon.amount;
        const res = String(this.coupon.amount).split('.');
        if (res.length === 1 || res[1].length < 3) {
          value = value.toFixed(2);
        }
        return `${value} AZN`;
      }
      return '';
    },
    getReceiptTitle() {
      return this.coupon.sender ? this.$t('label.coupon.sent') : this.$t('label.coupon.coming');
    },
    getMaskedStr() {
      return this.coupon.couponId.replace(/(.{8})(.{4}).{20}/, '$1-$2...');
    },
    getPaymentSource() {
      if (this.coupon.sender === false && this.coupon.status === 'ACCEPTED') {
        return {
          content: this.$t('label.coupon'),
          logo: '$coupon',
        };
      }
      return {
        content: this.$t('label.terminal'),
        logo: '$terminal',
      };
    },
    getCouponStatus() {
      if (this.coupon.sender) {
        switch (this.coupon?.status) {
          case 'PENDING':
            return {
              title: this.$t('label.coupon.pending.title'),
              icon: '$gift-pending',
              status: this.$t('label.coupon.pending.status'),
            };
          case 'READY':
            return {
              title: this.$t('label.coupon.sent'),
              icon: '$gift-ready',
              status: this.$t('label.success'),
            };
          case 'ACCEPTED':
            return {
              title: this.$t('label.coupon.accepted.title'),
              icon: '$gift-success',
              status: this.$t('label.coupon.accepted.status'),
            };
          case 'REJECTED':
            return {
              title: this.$t('label.coupon.rejected.title'),
              icon: '$gift-error',
              status: this.$t('label.coupon.rejected.status'),
            };
          default:
            return {
              title: this.$t('label.coupon.reversed.title'),
              icon: '$gift-reverse',
              status: this.$t('label.coupon.reversed.status'),
            };
        }
      } else {
        switch (this.coupon?.status) {
          case 'READY':
            return {
              title: this.$t('label.coupon.coming'),
              icon: '$gift-ready',
              status: this.$t('label.success'),
            };
          case 'ACCEPTED':
            return {
              title: this.$t('label.coupon.ready.title'),
              icon: '$gift-success',
              status: this.$t('label.coupon.ready.status'),
            };
          default:
            return '';
        }
      }
    },
    getReceipt() {
      if (this.coupon) {
        const def = [
          [this.$t('misc.coupon.receipt.id'), this.getMaskedStr],
          [this.$t('misc.transfer.receipt.status'), this.getCouponStatus.status],
          [this.$t('misc.transfer.confirm.amount'), this.getAmount],
          [this.coupon.sender ? this.$t('misc.coupon.receipt.recipient.gift') : this.$t('label.coupon.sender.gift'), this.coupon.sender ? this.coupon.reciverPhoneNumber : this.coupon.phoneNumber],
          [
            this.coupon.sender ? this.$t('misc.coupon.receipt.submitted.date') : this.$t('filters.date'),
            moment(this.coupon.sendDate).format('DD.MM.YYYY, HH:mm'),
          ],
          [this.$t('misc.coupon.receipt.message'), this.coupon.description],
          [this.$t('misc.payment.source'), this.getPaymentSource],
        ];
        return def;
      }
      return '';
    },
  },
  async mounted() {
    await this.loadReceipt();
  },
  methods: {
    async loadReceipt() {
      const [response, error] = await CouponRequests.getCouponInfo(this.couponId);
      if (error) {
        this.$notify.error(error.errorDetails[0].message);
        this.showAddFavorite = false;
      }
      this.coupon = response.data;
    },
    async downloadReceipt() {
      this.isLoading = true;
      const req = {
        couponCode: this.coupon.couponId,
      };
      await CouponRequests.downloadReceipt(req, { responseType: 'blob' }).then((response) => {
        console.log(response);
        const url = window.URL.createObjectURL(response[0]);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'receipt';
        a.click();
        a.remove();
        setTimeout(() => window.URL.revokeObjectURL(url), 100);
      }).catch(() => {
        this.$notify.error(this.$t('server.error.internal'));
        this.isLoading = false;
      });
      this.isLoading = false;
    },
    async topUpCoupon() {
      this.isCouponTopUpLoading = true;
      const [response, error] = await PaymentRequests.topUp({
        couponId: this.coupon.couponId,
        paymentChannel: 'COUPON',
        amount: this.coupon.amount,
      });
      console.log('response', response);
      if (error) {
        this.$notify.error(error?.errorDetails[0].message);
      } else {
        this.$notify.success(this.$t('label.success'));
        this.isCouponTopUpLoading = false;
        this.isCouponTopUpCompleted = true;
      }
    },
    copyReceiptCode(code) {
      navigator.clipboard.writeText(code);
      this.$notify.success(this.$t('misc.receipt.code.copy'));
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/views/components/receipt.scss';
@import '@/styles/components/dialogModalButtons.scss';
@import '@/styles/components/appDialogModal.scss';
.receipt .receipt-wrapper__footer__item {
  max-width: 180px;
  height: auto;
  gap: 8px;
  .btn-title{
    height: auto;
  }
  &:last-of-type {
    max-width: 180px;
  }
}

</style>
