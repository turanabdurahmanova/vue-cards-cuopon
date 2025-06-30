<template lang="pug">
.card-saved-list
  .card-saved-list_item(v-if="getCards.length>0" v-for="card in getCards" :key="card.label")
    .card-saved-list_item_left
      v-img(v-if="card?.bank" :src="card?.bank?.logoPath").card-saved-list_item_left_icon
      v-icon(v-else).card-saved-list_item_left_icon {{card?.binType ==  "VISA" ? '$visa-card' : '$master-card'}}
      .card-saved-list_item_left_title
        span.card-saved-list_item_left_title_span {{ card.label }}
        span.card-saved-list_item_left_title_span {{ card.cardNo }}
    .card-saved-list_item_right(v-if="!$vuetify.breakpoint.mobile")
      v-tooltip(top)
        template(v-slot:activator="{ on, attrs }")
          v-btn.card-saved-list_item_right_btn(id="open_edit_card_dialog" icon @click="openEditCardDialog(card)"
              v-bind="attrs" v-on="on")
            v-icon $edit-card
        span(class='tooltip-content') {{ $t('action.edit_label') }}
      v-tooltip(top content-class="customRed")
        template(v-slot:activator="{ on, attrs }")
          v-btn.card-saved-list_item_right_btn(id="open_delete_card_dialog" icon @click="openDeleteCardDialog(card)"
              v-bind="attrs" v-on="on")
            v-icon $delete
        span {{ $t('action.delete') }}
    v-menu(v-else left offset-y)
      template(v-slot:activator="{ on }")
        v-icon(v-on="on") mdi-dots-vertical
      v-list(style="min-width:190px").py-2
        v-list-item(v-for="action in cardActions" style="height:36px !important;"
        :key="action.title" @click.stop="action.method(card)").py-2.px-5
          v-list-item-icon.my-0.mr-3.align-self-center
            v-icon {{ action.icon }}
          v-list-item-title {{ $t(action.title) }}
  component(
    :is="currentForm"
    :is-modal-visible.sync='isModalVisible'
    :cards-list.sync="cards"
    :card-label.sync="label"
    :card="opCard")
</template>
<script>
import AppDialogModal from '@/components/AppDialogModal';
import EditForm from '@/views/dashboard/pages/cards/modal/edit-form';
import CardOrderSuccess from '@/views/dashboard/pages/cards/modal/card-order-success';
import DeleteForm from '@/views/dashboard/pages/cards/modal/delete-form';
import AppGoBackIcon from '@/components/AppGoBackIcon';

const types = {
  4: {
    type: 'visa',
    logo: '$visa-card',
  },
  5: {
    type: 'master',
    logo: '$master-card',
  },
  6: {
    type: 'master',
    logo: '$master-card',
  },
};

export default {
  name: 'SavedCards',
  components: {
    EditForm,
    DeleteForm,
    CardOrderSuccess,
    AppDialogModal,
    AppGoBackIcon,
  },
  props: {
    cards: {
      type: Array,
      required: true,
    },
    isCobrandCard: {
      type: Boolean,
      default: false,
    },
    isSavedCard: {
      type: Boolean,
      default: false,
    },
    isCardOrderInfo: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      deleteCardDialog: false,
      editCardDialog: false,
      isModalVisible: false,
      currentForm: '',
      opCard: {},
      label: '',
      cardActions: [
        {
          icon: '$edit-card-active',
          title: 'misc.saved.update.title',
          method: this.openEditCardDialog,
        },
        {
          icon: '$delete-active',
          title: 'action.remove',
          method: this.openDeleteCardDialog,
        },
      ],
    };
  },
  computed: {
    getCards() {
      return this.cards.map((card) => ({ ...card, cardLogo: types[card.cardNo[0]].logo }));
    },
  },
  methods: {
    openDeleteCardDialog(card) {
      this.opCard = card;
      this.label = card.label;
      this.currentForm = 'DeleteForm';
      this.isModalVisible = true;
    },
    openEditCardDialog(card) {
      this.opCard = card;
      this.label = card.label;
      this.currentForm = 'EditForm';
      this.isModalVisible = true;
    },
    openCardOrderSuccessDialog() {
      this.currentForm = 'CardOrderSuccess';
      this.isModalVisible = true;
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/views/cards.scss';
@import '@/styles/components/empty.scss';
.card-saved-info_left_title {
  margin-bottom: -10px;
}
.alert{
  padding: 16px 24px;
  border-radius: 10px;
  background: #FFF4ED;
  margin-bottom: 32px;
  cursor: pointer;
  &__wrapper{
    display: flex;
    align-items: center;
    justify-content: space-between;
    &__content{
      color: #FE8C4C;
      font-size: 16px;
      font-weight: 500;
      line-height: 150%; /* 24px */
    }
  }
}
@media (max-width: $xsBreakpoint) {
  .alert__wrapper__content{
    max-width: 255px;
    width: 100%;
  }
  .card-saved-info_left_title{
    margin-bottom: -4px;
  }
  .card-saved-list.onlySavedCard{
    background-color: #F8F8F8;
    padding: 20px;
    .card-saved-list_item{
      border-radius: 6px;
      background: #FFF;
      padding: 16px;
      .card-saved-list_item_left_title_span{
        font-size: 14px;
        &:last-of-type{
          font-size: 12px;
        }
      }
    }
  }
  .card-saved-list.bothCard{
    padding: 0 16px;
    margin-top: 24px;
  }

  // saved cards without cobrand
  .card-saved-list_item {
    padding: 16px;
    border-radius: 6px;
  }
  .card-saved-list_item_left_title_span {
    font-size: 14px;
    &:last-of-type {
      font-size: 12px;
    }
  }
}
</style>
