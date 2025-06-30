<template lang="pug">
    app-dialog-modal(
      :is-visible.sync="isVisible"
      :max-width="400"
      :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
      toolbar-top-position=".3125rem"
      modal-title-icon="$error-empty"
      modal-heading-margin-bottom="2rem"
      :modal-title="$t('misc.notify.general.error')"
      modal-title-font-size="1rem"
      :modal-description="$t('misc.notify.general.error.desc')")
      template(#modalBody)
        div.dialog_modal_button_group
          v-btn(depressed @click="isVisible = false").dialog_modal_button.rounded-lg
            | {{ $t('action.close') }}
    </template>

<script>

import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'ErrorModal',
  components: {
    AppDialogModal,
  },
  props: {
    isErrorModalVisible: {
      type: Boolean,
      default: false,
    },
  },

  computed: {
    isVisible: {
      get() {
        return this.isErrorModalVisible;
      },
      set(value) {
        this.$emit('update:isErrorModalVisible', value);
      },
    },
  },

  methods: {
    close() {
      this.isVisible = false;
    },
  },

};
</script>
