<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
  :max-width="400"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$pin-change"
  icon-size="65px"
  modal-heading-margin-bottom="2rem"
  :modal-title="$t('label.reset.pin.title')"
  modal-title-font-size="1rem"
  :modalDescription="$t('label.reset.pin.desc')")
  template(#modalBody)
    div.dialog_modal_button_group
      v-btn(id="pin_change_cinfirm" depressed @click="isVisible = false").dialog_modal_button.primary {{ $t('action.try') }}
      v-btn(id="pin_change_cancel" depressed).dialog_modal_button
        | {{ $t('action.cancel') }}
</template>

<script>
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
  },
  data() {
    return {
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
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
@import '@/styles/components/appDialogModal.scss';
.dialog_modal_button_group{
  flex-direction: column;
  gap: 1rem;
}
.dialog_modal_button.primary{
  font-size: 14px;
}
</style>
