<template lang="pug">
.wrapper
  portal(to="layout-top")(v-if="!$vuetify.breakpoint.mobile")
    v-row.pt-5(align="center")
      v-col.d-flex.align-center
        AppGoBackIcon
        h1.goback_title {{$t('label.card.order')}}
      v-col
        v-breadcrumbs(:items="items" class="pa-0 justify-end")
          template(v-slot:divider)
            v-icon mdi-chevron-right
  .goback(v-else)
    AppGoBackIcon
    h1.goback_title {{$t('label.card.order')}}
  .card-order-form
      CardOrderForm
</template>

<script>
import CardOrderForm from '@/views/dashboard/pages/cards/components/card-order-form';

import AppGoBackIcon from '@/components/AppGoBackIcon';

export default {
  name: 'CobrandOrder',
  components: {
    CardOrderForm,
    AppGoBackIcon,
  },
  data() {
    return {
      registerForm: true,
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
    };
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/cards.scss';
.card-order-form{
    padding: 70px 0;
    border-radius: 20px;
    background: #FFF;
}
@media (max-width: 650px) {
  .card-order-form{
    padding: 0px;
    width: auto;
    margin-top: 30px;
    border-radius: 0;
  }
}
</style>
