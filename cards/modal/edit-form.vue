<template lang="pug">
  div
    app-dialog-modal(
      :is-visible.sync="isVisible"
      :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
      toolbar-height="10px"
      modal-title-icon="$edit"
      :modal-title="$t('misc.card.update.title')")
      template(#modalBody)
        v-form(ref="editCard" @submit.prevent="saveCardLabel")
          v-text-field.favorite_name_field.card_label_field.py-8(
            v-if="card"
            v-model="label"
            :rules="isCardLabelValid"
            hide-details="auto"
            maxLength="25"
            clearable
            outlined
            required)
          div.dialog_modal_button_group
            v-btn(id="edit_card_cancel" depressed @click="isVisible = false").dialog_modal_button.rounded-lg
              | {{ $t('action.cancel') }}
            v-btn(id="edit_card_change" depressed type="submit" color="primary").dialog_modal_button.rounded-lg
              | {{ $t('action.confirm') }}
    component(
      :is="currentForm"
      :is-error-modal-visible.sync='isErrorModalVisible')
</template>

<script>
import AppDialogModal from '@/components/AppDialogModal';
import { ValidationRequired } from '@/rules/required';
import { PaymentRequests } from '@/services/http/payment-requests';
import ErrorModal from '@/views/dashboard/pages/cards/modal/error-modal';

export default {
  name: 'EditForm',
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
    cardLabel: {
      type: String,
      default: '',
    },
    isModalVisible: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      isCardLabelValid: [ValidationRequired()],
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
    label: {
      get() {
        return this.cardLabel;
      },
      set(value) {
        this.$emit('update:cardLabel', value);
      },
    },
  },
  methods: {
    async saveCardLabel() {
      if (this.$refs.editCard.validate()) {
        const [, error] = await PaymentRequests.updateSavedCardLabel(this.card.id, { label: this.label });

        if (!error) {
          this.cards = this.cards.map((card) => {
            if (card.id === this.card.id) {
              card.label = this.label;
            }
            return card;
          });
          this.$notify.success(this.$t('label.success.card.name'));
          this.isVisible = false;

        } else {
          this.isErrorModalVisible = true;
          this.currentForm = 'ErrorModal';
        }
      }
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
</style>
