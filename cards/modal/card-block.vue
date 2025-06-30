<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
  modal-class = "card-block"
  :max-width="400"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$card-block"
  modal-heading-margin-bottom="2rem"
  :modal-title="$t('label.block.card.title')"
  modal-title-font-size="1rem"
  icon-size = "2.18rem"
  :modal-description="$t('label.block.card.desc')")
  template(#modalBody)
    .dialog_modal_button_group
      v-btn(id="card_block_confirm" depressed @click="blockCard" color="error" :class="isLoading ? 'disabled' : ''").dialog_modal_button {{ $t('action.block') }}
        v-progress-circular(:style="[ !isLoading ? { height: 0 }: { height: '20px'}]" :width="2" :size="20" color="white" indeterminate)
      v-btn(id="card_block_cancel" depressed @click="isVisible = false").dialog_modal_button {{ $t('action.search.cancel') }}
</template>

<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardBlock',
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
      await MpayCobrandRequests.isLock({ cardId: this.card.cardId, isLock: true }).then((response) => {
        this.$notify.success(response[0].data.statusText);
      }).catch((error) => {
        this.$notify.error(error[0].data.statusText);
      }).finally(() => {
        this.$emit('clicked-islock', true);
        this.isLoading = false;
        this.isVisible = false;
      });
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
.dialog_modal_button_group .dialog_modal_button:first-of-type :deep(.v-btn__content) {
  gap: 10px;
  padding-left: 30px;
}
</style>
