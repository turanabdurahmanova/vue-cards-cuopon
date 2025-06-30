<template lang="pug">
  .cobrand-transfer
    .cobrand-transfer_alert
      slot(v-if="step === 1")
      v-alert.caption(v-if="step === 1" color="#FEF5EC" icon="mdi-information-outline")
        span {{ $t('misc.transfer.to_card.info') }}
    v-form.cobrand-transfer_form(v-show="step === 1" ref="cobrandForm" v-model="form" :disabled="loading")
      v-text-field.card-input(
        outlined
        :label="$t('label.target_card')"
        placeholder="1234 5678 9011 2345"
        persistent-placeholder
        :rules="rules.card"
        required
        :loading="isBinCheckInProgress"
        :disabled="isBinCheckInProgress"
        v-model="card.masked"
        v-maska:[cardOptions]="card"
        @maska="checkBin"
        :error-messages="binError")
        template(v-slot:append)
          v-icon.default-logo(v-if="!binFound") $transfer-credit-card
          img.bank-logo(v-else :src="logo")
      AmountInput(v-model="amount" amountInputLabel="Köçürmə məbləği")
      v-text-field.card-input.mpay-icon(
      outlined
      :label="$t('label.transfer.source')"
      prepend-inner-icon="$mpay-logo"
      v-model="cardNumber")
    div(v-show="step === 2")
      .subtitle-1.font-weight-medium.mb-5 {{ $t('misc.transfer.confirm') }}
      .d-flex.flex-column(style="gap:1em;")
        div(v-if="binFound!==''")
          span.transfer-detail__title Bank
          br
          span.transfer-detail__content {{ binFound }}
        div
          span.transfer-detail__title {{$t('label.target_card')}}
          br
          span.transfer-detail__content {{ card.masked }}
        div
          span.transfer-detail__title {{ $t('misc.transfer.confirm.amount') }}
          br
          span.transfer-detail__content {{ this.amount }} {{$t('misc.transfer.confirm.amount.currency')}}
        div
          span.transfer-detail__title {{ $t('misc.transfer.confirm.commission') }}
          br
          span.transfer-detail__content 0.60 {{$t('misc.transfer.confirm.amount.currency')}}
        div
          span.transfer-detail__title {{ $t('misc.transfer.confirm.all_amount') }}
          br
          span.transfer-detail__content {{ this.total }} {{$t('misc.transfer.confirm.amount.currency')}}
        div
          v-text-field.card-input.mpay-icon(
          outlined
          :label="$t('label.transfer.source')"
          prepend-inner-icon="$credit-card-active"
          v-model="cardNumber")
    .align-end
      hr.mb-4(style="border: 1px solid #eaeaea")
      v-row.text-right
        v-col.d-flex.justify-end
          v-btn(
            @click="back"
            id="transfer_card_form_back"
            x-large
            class="payment-form-btn rounded-lg mr-4 default-disable btn-back"
            depressed) Geriyə
          v-btn(
            id="transfer_card_form_continue"
            :disabled="!form"
            v-if="step === 1"
            @click="transferContinue"
            :loading="loading"
            color="primary"
            x-large
            class="payment-form-btn rounded-lg"
            depressed) {{ $t('action.continue') }}
          v-btn(
            id="transfer_card_form_confirm"
            v-if="step === 2"
            @click="transfer"
            :loading="loading"
            color="primary"
            x-large
            class="payment-form-btn rounded-lg"
            depressed) {{ $t('action.confirm') }}
</template>
<script>
import { ValidationRequired } from '@/rules/required';
import { PaymentRequests } from '@/services/http/payment-requests';
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import { mapGetters, mapActions } from 'vuex';
import AppPaymentSource from '@/components/AppPaymentSource';
import AppGoBackIcon from '@/components/AppGoBackIcon';

export default {
  name: 'CobrandForm',
  components: {
    AppPaymentSource,
    AppGoBackIcon,
  },
  data: () => ({
    props: {
      bottom: true, offsetY: true, maxHeight: 500,
    },
    card: {
      masked: '',
      unmasked: '',
      completed: false,
    },
    cardOptions: {
      eager: true,
      mask: '#### #### #### ####',
    },
    rules: {
      card: [ValidationRequired(), (v) => (v && v.length === 19)],
      select: [ValidationRequired()],
    },
    step: 1,
    loading: false,
    form: true,
    amount: null,
    isSaved: false,
    checkedBin: '',
    isBinCheckInProgress: false,
    binError: '',
    binFound: '',
    productId: '',
    logo: '',
    cardNumber: null,
    mpayCard: null,
  }),
  computed: {
    ...mapGetters('Payment', ['getWalletBalance', 'getWalletAmountRaw']),
    total() {
      return this.amount * 1 + 0.60;
    },
    isMobile() {
      return this.$vuetify.breakpoint.mobile;
    },
  },
  async mounted() {
    await MpayCobrandRequests.getMpayCobrandCard().then((response) => {
      this.mpayCard = response[0].data?.response[0];
      this.cardNumber = this.getMaskedCard(this.mpayCard.cardNumber);
    }).catch(() => {});
  },
  methods: {
    ...mapActions('Favorite', ['setFavorite']),
    getMaskedCard(cardNo) {
      if (cardNo) {
        // Extract the first 6 digits and the last 4 digits
        const firstSix = cardNo?.substring(0, 6);
        const lastFour = cardNo?.substring(cardNo.length - 4);

        // Mask the digits in between with '*'
        const maskedDigits = cardNo?.substring(6, cardNo.length - 4).replace(/[a-zA-Z0-9]/g, '*');
        // Concatenate the masked parts
        const text = `${firstSix}${maskedDigits}${lastFour}`;
        return `MPAY kartım ${text.slice(0, 4)} ${text.slice(4, 8)} ${text.slice(8, 12)} ${text.slice(12, 16)}`;
      }

      return '';

    },
    async checkBin(e) {
      if (e.detail.completed && this.checkedBin !== e.detail.unmasked) {
        this.isBinCheckInProgress = true;
        await PaymentRequests.binCheck(this.card.unmasked).then((response) => {
          if (response[0].data.binInfo) {
            this.binFound = response[0].data.binInfo?.bankName;
            this.logo = response[0].data.binInfo?.logoPath;
          }
        }).catch(() => {
          this.binError = this.$t('validation.card.number.invalid');
        }).finally(() => {
          this.isBinCheckInProgress = false;
        });
      }
      this.checkedBin = e.detail.unmasked;
    },
    async transfer() {
      if (!this.$refs.cobrandForm.validate()) {
        return;
      }
      this.loading = true;
      const req = {
        sourceCardId: this.mpayCard.cardId,
        destinationCard: this.card.unmasked,
        amount: parseFloat(this.amount),
        description: '',
      };

      await MpayCobrandRequests.cardTocard(req).then(() => {
        this.$router.push('/cards');
        // this.$router.push({ name: 'transfer-receipt', params: { id: this.mpayCard.cardId } });
      }).catch((error) => {
        this.$notify.error(error.errorDetails[0].message);
      });
      this.loading = false;
    },
    async transferContinue() {
      if (!this.$refs.cobrandForm.validate()) {
        return;
      }

      const numValue = Number(this.amount);

      if (this.getWalletAmountRaw - numValue < 0) {
        this.$notify.error(this.$t('validation.error.not-enough-ballance'));
        return;
      }
      this.step = 2;
    },
    back() {
      if (this.step === 2) {
        this.step = 1;
      }
      this.$router.push({ name: 'cards' });
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/transfers/card-form.scss';
.cobrand-transfer_form{
  max-width: 400px;
  width: 100%;
}
.cobrand-transfer_alert{
  max-width: 400px;
  width: 100%;
}
</style>
