<template lang="pug">
app-dialog-modal(
    :is-visible.sync="isVisible"
    :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
    :max-width="400"
    toolbar-height="10px"
    toolbar-top-position=".3125rem"
    modal-title-icon="$deposit-deactive"
    modal-heading-margin-bottom="2rem"
    :modal-title="$t('label.deposit.deactivate.title')"
    modal-title-font-size="1rem"
    icon-size = "4rem"
    :modal-description="$t('label.deposit.deactivate.desc')")
    template(#modalBody)
      .dialog_modal_button_group
        v-btn(id="card_unblock_cancel" depressed @click="unBlockCard" color="primary" :class="isLoading ? 'disabled' : ''").dialog_modal_button {{ $t('action.deposit.deactivate') }}
          .progress-box(v-if="isLoading")
            v-progress-circular(:width="2" :size="20" color="white" indeterminate)
        v-btn(id="card_unblock_confirm" depressed @click="isVisible = false").dialog_modal_button  {{ $t('action.search.cancel') }}
</template>
<script>

// import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardUnblock',
  components: {
    AppDialogModal,
  },
  props: {
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
      isLoading: false,
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
    async unBlockCard() {
      this.isLoading = true;
      // await MpayCobrandRequests.isLock({ cardId: this.card.cardId, isLock: false }).then((response) => {
      //   this.$notify.success(response[0].data.statusText);
      // }).catch((error) => {
      //   this.$notify.error(error[0].data.statusText);
      // }).finally(() => {
      //   this.$emit('clicked-islock', false);
      //   this.isLoading = false;
      // });
      this.isVisible = false;
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
.dialog_modal_button_group {
  flex-direction: column;
  gap: 1rem;
}
</style>
