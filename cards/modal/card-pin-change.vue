<template lang="pug">
  app-dialog-modal(
    :is-visible.sync="isVisible"
    :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
    toolbar-height="10px"
    modal-title-icon="$pin-change"
    :modal-title="$t('label.change.pin.title')")
    template(#modalBody)
      v-form(ref="pinSetRef" @submit.prevent="pinSetSubmit")
        v-text-field(
          v-model="pin"
          :rules="rules.pin"
          :placeholder="$t('label.change.pin.new')"
          hide-details="auto"
          maxLength="25"
          type="number"
          :min="0"
          clearable
          outlined
          required)
        v-text-field(
          v-model="pinConfirm"
          :rules="rules.confirm_pin"
          :error-messages="pinNotMatch"
          @keyup="checkPinMatch"
          type="number"
          :min="0"
          :placeholder="$t('label.change.pin.new.reenter')"
          hide-details="auto"
          maxLength="25"
          clearable
          outlined
          required)
        div.dialog_modal_button_group
          v-btn(id="edit_card_cancel" depressed @click="isVisible = false").dialog_modal_button.rounded-lg
            | {{ $t('action.cancel') }}
          v-btn(id="edit_card_change" :disabled="resetPinConfirmForm" depressed type="submit" color="primary").dialog_modal_button.rounded-lg {{ $t('action.confirm') }}
            .progress-box(v-show="isLoading")
              v-progress-circular(:width="2" :size="20" color="white" indeterminate)
</template>

<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import { ValidationPattern } from '@/rules/pattern';
import { ValidationRequired } from '@/rules/required';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardPinChange',
  components: {
    AppDialogModal,
  },
  props: {
    isModalVisible: {
      type: Boolean,
      default: false,
    },
    card: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      pin: '',
      pinConfirm: '',
      pinNotMatch: '',
      resetPinConfirmForm: true,
      isLoading: false,
      rules: {
        pin: [ValidationRequired(), ValidationPattern('(.{4,})', 'Simvol say 4 olmali')],
        confirm_pin: [ValidationRequired(), (value) => {
          if (value === this.pin) {
            return true;
          }

          return 'Pin uyğun deyil';
        }],
      },
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
  methods: {
    checkPinMatch() {
      if (this.pin !== this.pinConfirm && this.pinConfirm !== '') {
        this.pinNotMatch = 'Pin uyğun deyil';
      } else {
        this.pinNotMatch = '';
        this.resetPinConfirmForm = false;
      }
    },
    async pinSetSubmit() {
      if (this.$refs.pinSetRef.validate()) {
        this.isLoading = true;
        const pinSetObj = {
          fin: '',
          pin: this.pin,
          cardId: this.card.cardId,
        };
        await MpayCobrandRequests.pinSet(pinSetObj).then(() => {
          this.isVisible = false;
        }).catch(() => {
          this.$notify.error(this.$t('server.error.internal'));
        }).finally(() => {
          this.$notify.success(this.$t('misc.notify.general.success'));
          this.isLoading = false;
          this.$refs.pinSetRef.reset();
          this.resetPinConfirmForm = true;
        });
      }
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
@import '@/styles/components/appDialogModal.scss';
.v-form{
  padding-top: 32px;
}
.dialog_modal_button_group{
  padding-top: 32px;
}
.v-text-field.v-text-field--enclosed.v-text-field--outlined{
  padding-top: 0!important;
  color: #ACACAC;
  font-size: 14px;
  font-weight: 400;
  line-height: 125%; /* 17.5px */
  &:first-of-type{
    padding-bottom: 16px;
  }
}
</style>
