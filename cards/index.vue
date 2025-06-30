<template lang="pug">
    AppSheetComponent
        portal(to="layout-top")(v-if="!$vuetify.breakpoint.mobile")
            v-row(align="center")
                v-col.d-flex.align-center
                    AppGoBackIcon
                    h1.goback_title {{$t('label.my.cards')}}
                v-col
                    v-breadcrumbs(:items="items" class="pa-0 justify-end")
                    template(v-slot:divider)
                        v-icon mdi-chevron-right
        .goback(v-else)
            AppGoBackIcon
            h1.goback_title {{$t('label.my.cards')}}
        .loader(v-if="isLoading")
            app-loader
        .card-saved(v-else-if="isSavedCard")
            SavedCards(:cards ="cards")
        .empty-page(v-else)
            .empty-page_wrapp
              v-img.empty-page_img(src="@/assets/empty-card.png")
              span.empty-page_txt {{ $t('misc.saved.index.empty') }}
              v-btn.empty-page_btn(id="cards_go_dashboard_root" :to="{name: 'dashboard-root'}" depressed) {{ $t('action.go_dashboard') }}
</template>
<script>
import AppLoader from '@/components/AppLoader';
import AppGoBackIcon from '@/components/AppGoBackIcon';
import AppSheetComponent from '@/components/AppSheetComponent';
import SavedCards from '@/views/dashboard/pages/cards/components/saved-cards';
import { PaymentRequests } from '@/services/http/payment-requests';

export default {
  name: 'IndexPage',
  components: {
    AppLoader,
    AppGoBackIcon,
    AppSheetComponent,
    SavedCards,
  },
  data() {
    return {
      isSavedCard: false,
      isLoading: true,
      items: [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          href: '',
        },
        {
          text: this.$t('label.my.cards'),
          disabled: true,
          href: '',
        },
      ],
      cards: [],
    };
  },
  async mounted() {
    const [response] = await PaymentRequests.savedCards();

    if (response) {
      this.cards = response?.data;
      this.isSavedCard = this.cards.length > 0;
      this.isLoading = false;
    }
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/views/cards.scss';
@import '@/styles/components/empty.scss';
@media (max-width: 1264px) {
    .goback {
        padding: 0;
        margin-bottom: 24px;
    }
    .card-saved {
        padding: 0;
    }
}

</style>
