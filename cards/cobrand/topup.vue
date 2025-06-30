<template lang="pug">
div.top-up-page
  AppBreadCrumbs(
    pageName="action.top_up"
    :crumbs="items"
  )
  v-sheet.pa-8(rounded="xl" height="80vh" color="white"
    class="d-flex flex-column justify-space-between")
    v-row.pb-2(v-if="$vuetify.breakpoint.mobile" align="center")
      v-col
        AppGoBackIcon
      v-col.text-center
        div.text-h6 {{$t('action.top_up')}}
      v-col.d-flex.justify-end
    v-form(v-if="step === 1" ref="topup" v-model="topUpForm" :disabled="loading")
      v-text-field.card-input.mpay-icon(
      outlined
      :label="$t('label.target_card')"
      prepend-inner-icon="$mpay-logo"
      v-model="cardNumber")
      AmountInput(v-model="amount")
      app-payment-source(
        :payment-method.sync="selected"
        :items="cards"
        item-text="cardNo"
        item-value="cardToken"
        :menu-props="props"
        :is-mobile-screen="isMobile"
        :is-disabled="false")
    div(v-if="step === 2")
      .d-flex.flex-column(style="gap:1em;")
        .subtitle-1.font-weight-medium {{ $t('misc.transfer.confirm') }}
        div
          span.topup-detail__title {{ $t('misc.transfer.confirm.amount') }}
          br
          span.topup-detail__content {{ this.resultAmount?.amount }} AZN
        div
          span.topup-detail__title {{ $t('misc.transfer.confirm.commission') }}
          br
          span.topup-detail__content {{ this.resultAmount?.commission }} AZN
        div
          span.topup-detail__title {{ $t('misc.transfer.confirm.all_amount') }}
          br
          span.topup-detail__content {{ this.resultAmount?.totalAmount }} AZN
        app-payment-source(:payment-method.sync="selected" :items="cards" :is-disabled="true")
    .align-end
      hr.mb-4(style="border: 1px solid #eaeaea")
      v-row.text-right
        v-col.d-flex.justify-end
          v-btn(
          id="transfer_card_form_back"
          x-large
          class="payment-form-btn rounded-lg mr-4 default-disable btn-back"
          depressed
          :to="'/cards'") Geriyə
          v-btn(
            id="top_up_confirm"
            v-if="step === 2"
            @click="topUp"
            :loading="loading"
            color="primary"
            x-large
            class="payment-form-btn rounded-lg"
            depressed) {{ $t('action.confirm') }}
</template>
<script>
import AppBreadCrumbs from '@/components/AppBreadCrumbs';
import { PaymentRequests } from '@/services/http/payment-requests';
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppPaymentSource from '@/components/AppPaymentSource';

export default {
  name: 'CobrandTopUp',
  components: {
    AppBreadCrumbs,
    AppPaymentSource,
  },
  data() {
    return {
      props: {
        bottom: true, offsetY: true, maxHeight: 500,
      },
      step: 1,
      loading: false,
      topUpForm: true,
      amount: null,
      selected: null,
      items: [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          exact: true,
          to: {
            name: 'dashboard-root',
          },
        },
        {
          text: this.$t('action.top_up'),
          disabled: true,
          href: '',
        },
      ],
      cards: [],
      cardNumber: null,
      mpayCard: null,
    };
  },
  computed: {
    isMobile() {
      return this.$vuetify.breakpoint.mobile;
    },
  },
  async mounted() {
    const [response] = await PaymentRequests.savedCards();
    const [uberResponse] = await PaymentRequests.uberParkList();

    const savedCards = response.data.map((r) => {
      return {
        ...r,
        type: 'SAVED_CARD',
      };
    });

    const uberParks = uberResponse.data.map((r) => {
      return {
        type: 'UBER',
        id: r.id,
        icon: '$uber-icon',
        label: r.name,
        cardNo: null,
        cardToken: r.id,
      };
    });

    this.cards = [
      ...uberParks,
      ...[{ header: this.$t('label.saved_cards') }],
      ...savedCards,
      ...[
        {
          divider: true,
        },
        {
          type: 'NEW_CARD',
          label: this.$t('label.pay_with_new_card'),
          icon: '$credit-card-active',
        },
      ],
    ];

    this.selected = {
      type: 'NEW_CARD',
      label: this.$t('label.pay_with_new_card'),
      icon: '$credit-card-active',
    };

    await MpayCobrandRequests.getMpayCobrandCard().then((res) => {
      this.mpayCard = res[0].data?.response[0];
      this.cardNumber = this.getMaskedCard(this.mpayCard.cardNumber);
    }).catch(() => {});

  },
  methods: {
    async topUp() {
      if (!this.$refs.topup.validate()) {
        return;
      }
      this.loading = true;

      const cardId = this.selected.type === 'SAVED_CARD' || this.selected.type === 'UBER' ? { cardId: this.selected.id } : {};
      const cardTopUpBody = {
        paymentChannel: this.selected.type,
        amount: this.amount,
        ...cardId,
      };

      const uberTopUpBody = {
        paymentChannel: this.selected.type,
        amount: this.amount,
        sourceId: this.selected.id,
      };

      const [response, error] = await PaymentRequests.topUp(this.selected.type === 'UBER' ? uberTopUpBody : cardTopUpBody);
      this.loading = false;

      if (error) {
        this.$notify.error(error?.errorDetails[0].message);
      } else {

        // eslint-disable-next-line no-lonely-if
        if (this.selected.type === 'SAVED_CARD') {
          this.$router.push({ name: 'top-up-receipt', params: { id: response.data.orderUuid } });
        } else if (this.selected.type === 'UBER') {
          this.$router.push({ name: 'top-up-receipt', params: { id: response.data.orderUuid } });
        } else if (this.selected.type === 'NEW_CARD') {
          if (response?.data?.gatewayLink) {
            window.location = response.data.gatewayLink;
          } else {
            this.$router.push({ name: 'top-up-receipt', params: { id: response.data.orderUuid } });
          }
        }
      }
    },
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
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/top-up.scss';
@import '@/styles/views/transfers/card-form.scss';
</style>
