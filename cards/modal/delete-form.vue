<template lang="pug">
  div
    app-dialog-modal(
      :is-visible.sync="isVisible"
      :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
      toolbar-height="10px"
      toolbar-top-position=".3125rem"
      modal-title-icon="$warning"
      modal-heading-margin-bottom="2rem"
      :modal-title="$t('misc.saved.remove.title')"
      modal-title-font-size="1rem"
      :modal-description="$t('misc.saved.remove.desc')")
      template(#modalBody)
        div.dialog_modal_button_group
          v-btn(id="delete_card_cancel" depressed @click="isVisible = false").dialog_modal_button.rounded-lg
            | {{ $t('action.cancel') }}
          v-btn(id="delete_card_confirm" depressed @click="deleteCard" color="error").dialog_modal_button.rounded-lg
            | {{ $t('action.card.remove') }}
    component(
      :is="currentForm"
      :is-error-modal-visible.sync='isErrorModalVisible')
</template>

<script>
import AppDialogModal from '@/components/AppDialogModal';
import ErrorModal from '@/views/dashboard/pages/cards/modal/error-modal';
import { PaymentRequests } from '@/services/http/payment-requests';

export default {
  name: 'DeleteForm',
  components: {
    AppDialogModal,
    ErrorModal,
  },
  props: {
    cardsList: {
      type: Array,
      required: true,
    },
    card: {
      type: Object,
      required: true,
    },
    isModalVisible: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      currentForm: '',
      isErrorModalVisible: false,
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
    cards: {
      get() {
        return this.cardsList;
      },
      set(value) {
        this.$emit('update:cardsList', value);
      },
    },
  },
  methods: {
    async deleteCard() {
      const [, error] = await PaymentRequests.deleteSavedCard(this.card.id);
      if (!error) {
        this.cards = this.cards.filter((card) => card.id !== this.card.id);

        this.$notify.success(this.$t('label.success.card.delete'));

        this.isVisible = false;
      } else {
        this.isErrorModalVisible = true;
        this.currentForm = 'ErrorModal';
      }
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
</style>
