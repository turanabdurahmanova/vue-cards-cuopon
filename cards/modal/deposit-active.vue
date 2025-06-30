<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
  :max-width="400"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$deposit-active"
  modal-heading-margin-bottom="2rem"
  :modal-title="$t('label.deposit.activate.title')"
  modal-title-font-size="1rem"
  icon-size = "4rem"
  :modal-description="$t('label.deposit.activate.desc')")
  template(#modalBody)
    .dialog_modal_button_group
      v-btn(id="card_block_confirm" depressed @click="blockCard" color="primary" :class="isLoading ? 'disabled' : ''").dialog_modal_button  {{ $t('action.deposit.activate') }}
        .progress-box(v-if="isLoading")
          v-progress-circular(:width="2" :size="20" color="white" indeterminate)
      v-btn(id="card_block_cancel" depressed @click="isVisible = false").dialog_modal_button {{ $t('action.search.cancel') }}
</template>

<script>
// import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'DepositActive',
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
    async blockCard() {
      this.isLoading = true;
      // await MpayCobrandRequests.isLock({ cardId: this.card.cardId, isLock: true }).then((response) => {
      //   this.$notify.success(response[0].data.statusText);
      // }).catch((error) => {
      //   this.$notify.error(error[0].data.statusText);
      // }).finally(() => {
      //   this.$emit('clicked-islock', true);
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
