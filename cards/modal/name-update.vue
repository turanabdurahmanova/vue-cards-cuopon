<template lang="pug">
  app-dialog-modal(
    :is-visible.sync="isVisible"
    :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
    toolbar-height="10px"
    modal-title-icon="$edit"
    :modal-title="$t('label.rename.title')")
    template(#modalBody)
      v-form(ref="nameUpdateRef" v-model="updateNameForm" @submit.prevent="nameUpdateSubmit")
        v-text-field(
          v-model="name"
          :rules="isCardNameValid"
          :placeholder="$t('label.rename.title')"
          hide-details="auto"
          clearable
          outlined
          required)
        div.dialog_modal_button_group
          v-btn(id="edit_cobrandName_cancel" depressed @click="isVisible = false").dialog_modal_button
            | {{ $t('action.cancel') }}
          v-btn(id="edit_cobrandName_change" :disabled="!updateNameForm" depressed type="submit" color="primary").dialog_modal_button {{ $t('action.change')}}
            .progress-box(v-show="isLoading")
              v-progress-circular(:width="2" :size="20" color="white" indeterminate)
</template>

<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import { ValidationRequired } from '@/rules/required';
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CobrandNameUpdate',
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
      name: this.card.name || 'MPAY kart',
      isLoading: false,
      updateNameForm: true,
      isCardNameValid: [ValidationRequired()],
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
    async nameUpdateSubmit() {
      if (this.$refs.nameUpdateRef.validate()) {
        this.isLoading = true;
        await MpayCobrandRequests.update({ name: this.name }).then(() => {
          this.isVisible = false;
        }).catch(() => {
          this.$notify.error(this.$t('server.error.internal'));
        }).finally(() => {
          this.$notify.success(this.$t('misc.notify.general.success'));
          this.isLoading = false;
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
}
</style>
