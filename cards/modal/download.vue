<template lang="pug">
app-dialog-modal(
    :is-visible.sync="isVisible"
    :modal-padding="$vuetify.breakpoint.mobile ? '2rem 1rem' : '2.5rem'"
    modal-class = "card-block"
    :max-width="505"
    toolbar-height="10px"
    toolbar-top-position=".3125rem"
    modal-title-icon="$download-circle"
    modal-heading-margin-bottom="2rem"
    :modal-title="$t('label.mpay.download')"
    modal-title-font-size="1.125rem"
    modal-title-font-weight="600"
    icon-size = "2.18rem"
    :modal-description="$vuetify.breakpoint.mobile ? $t('action.mpay.download') : ''")
    template(#modalBody)
      .scan(v-if="!$vuetify.breakpoint.mobile")
        .scan_wrapp
          img.scan_wrapp_img(src="@/assets/qr.svg")
        .scan_content {{ $t('label.mpay.downoad.desc') }}
      v-btn(v-else id="download_mpay_app" :href='getMobileOperatingSystem' target="_blank" depressed color="primary").dialog_modal_button {{ $t('action.download') }}
</template>
<script>

import AppDialogModal from '@/components/AppDialogModal';

export default {
  name: 'AppDownload',
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
    getMobileOperatingSystem() {
      const userAgent = navigator.userAgent || window.opera;

      // Windows Phone must come first because its UA also contains "Android"
      if (/windows phone/i.test(userAgent)) {
        return 'https://apps.apple.com/us/app/mpay-e-wallet/id6450408051';
      }

      if (/android/i.test(userAgent)) {
        return 'https://play.google.com/store/search?q=mpay&c=apps';
      }

      // iOS detection from: http://stackoverflow.com/a/9039885/177710
      if (/iPad|iPhone|iPod/.test(userAgent) && !window.MSStream) {
        return 'https://apps.apple.com/us/app/mpay-e-wallet/id6450408051';
      }
      return 'unknown';
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/components/dialogModalButtons.scss';
.scan {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  &_wrapp {
    border-radius: 7px;
    background: #FFF;
    box-shadow: 0px 2.797px 48.946px 0px rgba(0, 0, 0, 0.22);
    padding: 14px;
    width: 120px;
    height: 120px;
    &_img{
      width: 100%;
      height: 100%;
    }
  }
  &_content {
    height: 120px;
    background: linear-gradient(90deg, #F1F1F1 29.11%, #FFF 100.9%);
    color:  #4B4B4B;
    font-size: 18px;
    font-weight: 500;
    line-height: 150%; /* 27px */;
    display: flex;
    align-items: center;
    width: calc(100% - 120px);
    padding-left: 35px;
    text-align: left;
  }
}

</style>
