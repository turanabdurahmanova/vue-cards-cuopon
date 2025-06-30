<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem 1rem' : '2.5rem'"
  :max-width="550"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$card-info-dialog"
  icon-size="65px"
  modal-heading-margin-bottom="1.5rem"
  :modal-title="$t('label.requisite.popup.title')"
  :is-snackbar-visible.sync = "snackbar")
  template(#modalBody)
    ul.card-dialog-list
      li.card-dialog-list_item(v-for="requisite in requisites")
        .card-dialog-list_item_left {{ $t(requisite.left) }}
        .card-dialog-list_item_right
          span.card-dialog-list_item_right_span {{requisite.right}}
          v-icon.cursor-pointer(@click="copyCode(requisite.right)") $copy
    .dialog_modal_button_group(@click="download" id="download_card_rekvizit")
      v-btn.dialog_modal_button.primary(depressed) {{ $t('action.download') }}
        .progress-box
          v-progress-circular(v-show="isLoading" :width="2" :size="20" color="white" indeterminate)
</template>

<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardRekvizit',
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
      requisites: [],
      isLoading: false,
      pdfData: null,
      snackbar: false,
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
    await MpayCobrandRequests.requisites(
      {
        iban: this.card.iban,
        userId: this.card.userId,
      },
    ).then((response) => {
      if (response[0].data.response.length) {
        const res = response[0].data.response[0];
        this.requisites = [
          { left: 'label.recipient.name', right: res.clientName },
          { left: 'label.buyer.account', right: res.iban },
          { left: 'label.receiving.bank', right: res.benBankName },
          { left: 'label.swift', right: res.benBankSwift },
          { left: 'label.vöen', right: res.voen },
          { left: 'label.bank.code', right: res.fad },
          { left: 'label.reporter.account', right: res.corrAccount },
        ];
      }
    }).catch((error) => {
      console.log(error);
    });
  },
  methods: {
    async download() {
      this.isLoading = true;
      const req = {
        iban: this.card.iban,
        userId: this.card.userId,
      };
      await MpayCobrandRequests.download(req).then((response) => {
        this.pdfData = `data:application/pdf;base64,${response[0].data}`;
        const a = document.createElement('a');
        a.href = this.pdfData;
        a.download = 'rekvizit';
        a.click();
        a.remove();
        setTimeout(() => window.URL.revokeObjectURL(this.pdfData), 100);
      }).catch(() => {
        this.$notify.error(this.$t('server.error.internal'));
        this.isLoading = false;
      }).finally(() => {
        this.isLoading = false;
      });
    },

    copyCode(code) {
      // this peace of code doesnt work on dev mode cause dev mod is in http without ssl , cause of that dev mode does not have ssl it works only with https
      navigator.clipboard.writeText(code);
      this.snackbar = true;
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
