<template lang="pug">
app-dialog-modal(
  :is-visible.sync="isVisible"
  :modal-padding="$vuetify.breakpoint.mobile ? '1.5rem' : '2.5rem'"
  :max-width="450"
  toolbar-height="10px"
  toolbar-top-position=".3125rem"
  modal-title-icon="$card-info-dialog"
  icon-size="65px"
  modal-heading-margin-bottom="1.5rem"
  :modal-title="$t('label.info.popup.title')")
  template(#modalBody)
    ul.card-info
      li.card-info_item
        span {{ $t('label.cardholder.name') }}
        span {{ card?.cardHolder }}
      li.card-info_item
        span {{ $t('label.card.number') }}
        span {{ card?.cardNumber}}
      li.card-info_item
        span {{ $t('label.expiry.date') }}
        span {{ card?.expiryDate}}
    .card-info_agree
      .card-info_agree_content
        img.card-info_agree_content_img(src="https://logo-mpay.s3.amazonaws.com/rabitabank.svg")
        span.card-info_agree_content_span  {{ $t('label.card.info.popup.desc') }}
      .card-info_agree_contact
        a.card-info_agree_contact_item(v-for="(item, i) in connects" :key="i" :href="item.href" target="_blank")
          v-icon.card-info_agree_contact_item_icon(size="20" v-text="item.icon")
          span.card-info_agree_contact_item_text {{item.text}}
</template>

<script>
import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'CardInfo',
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
  computed: {
    isVisible: {
      get() {
        return this.isModalVisible;
      },
      set(value) {
        this.$emit('update:isModalVisible', value);
      },
    },
    connects() {
      return [
        {
          icon: '$phone',
          text: '*4446',
          href: 'tel:*4446',
        },
        {
          icon: '$mail',
          text: 'contact@mpay.az',
          href: 'mailto:contact@mpay.az',
        },
      ];
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
@import '@/styles/components/appDialogModal.scss';
</style>
