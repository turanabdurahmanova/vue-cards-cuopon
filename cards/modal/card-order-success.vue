<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem 1rem' : '2.5rem'"
  :max-width="546"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$gift-success"
  icon-size="65px"
  modal-heading-margin-bottom="1.5rem"
  :modal-title="$t('label.card.order.success.title')")
  template(#modalBody)
    ul.card-dialog-list
      li.card-dialog-list_item(v-for="item in cardOrderList")
        span.card-dialog-list_item_left {{ $t(item.left) }}
        span.card-dialog-list_item_right {{item.right}}
</template>

<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import moment from 'moment';
import config from '@/config/variables';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardOrderSuccess',
  components: {
    AppDialogModal,
  },
  props: {
    isModalVisible: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      cardOrderList: [],
      errorCases: config.statusCases.error,
      pendingCases: config.statusCases.pending,
    };
  },

  computed: {
    isVisible: {
      get() {
        return this.isModalVisible;
      },
      set(value) {
        this.$emit('update:isModalVisible', value);
      },
    },
  },
  async mounted() {
    await MpayCobrandRequests.cardOrderList().then((response) => {
      const res = response[0].data;
      this.cardOrderList = [
        { left: 'label.name_surname', right: res.name.concat(' ').concat(res.surname) },
        // { left: 'label.card.type', right: res.paymentType },
        { left: 'label.card.type', right: this.$t('label.mpay.card') },
        { left: 'label.order.date', right: moment(res.createdAt).format('D.MM.YYYY') },
        { left: 'label.currency', right: res.currency },
        { left: 'label.branch.delivery', right: res.branchName || 'Rabitəbank, KOB Mərkəzi filialı' },
        { left: 'label.status', right: this.getReceiptInfo(res.status) },
      ];
    }).catch((error) => {
      console.log(error);
    });
  },
  methods: {
    getReceiptInfo(status) {
      if (this.pendingCases.includes(status)) {
        return this.$t('label.pending');
      }
      if (this.errorCases.includes(status)) {
        return this.$t('label.fail');
      }
      if (status === 'COMPLETED') {
        return this.$t('label.success');
      }
      return '';
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
@import '@/styles/components/appDialogModal.scss';
.dialog_modal_button.primary{
  height: 3.5rem !important;
  margin-top: 32px;
  font-size: 1rem;
}
</style>
