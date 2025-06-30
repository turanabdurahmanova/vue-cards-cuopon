<template lang="pug">
.wrapper
  portal(to="layout-top")(v-if="!$vuetify.breakpoint.mobile")
    v-row.pt-5(align="center")
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
  .loader(v-if="isLoading && !mpayCobrandCard")
    app-loader
  .card-saved(v-else-if="isSavedCard")
    .card-saved-info(v-if="cardOrderInfo?.status === 'ERROR' || cardOrderInfo === null")
      .card-saved-info_left
        h1.card-saved-info_left_title {{$t('label.card.order.title')}}
        p.card-saved-info_left_desc {{$t('label.card.order.desc')}}
        .card-saved-info_left_buttons
          v-btn.card-saved-info_left_button(:to="{name:'cobrand-order'}") {{$t('action.card.order')}}
          v-btn.card-saved-info_left_button(:to="{name:'cobrand-info'}") {{$t('action.more.detail')}}
      .card-saved-info_right
        img(src="@/assets/card-info.png")
    SavedCards(:cards = "cards" :isCardOrderInfo="isCardOrderInfo" :isSavedCard="isSavedCard")
  template(v-else)
    v-list.tabs
      v-list-item.tabs-item(v-for="(tab, index) in tabs" :key="tab.title" @click="selectTab(index)"  :class='{"tabs__selected": (index == selectedIndex)}') {{ $t(tab.title)}}
    .card.tabs__content(:class='{"isActive": (0 == selectedIndex)}')
      .card-rectangle
        .card-img
        .extract
          span.extract_text {{ $t('action.extract')}}
          v-icon.extract_icon $extract-file
        .card-action
          .mpay-card(:class="[ isLock ? 'isBlockBlur': '']")
            .mpay-card-visa
              .mpay-card-visa_left
                v-icon.mpay-card-visa_left_icon $transfer-credit-card
                span.mpay-card-visa_left_text  {{ $t('label.mpay.card')}}
              v-icon.mpay-card-visa_right $visa-white-card
            span.mpay-card_clientName(:class="[!isBalanceVisible ? 'blur-balance': '']") {{mpayCobrandCard?.clientName?.split(' ').slice(0, 2).join(' ')}}
            .mpay-card-balance
              .mpay-card-balance_left(:class="[!isBalanceVisible ? 'blur-balance': '']")
                span {{ mpayCobrandCard?.amount.toFixed(2) }}
                span AZN
              .mpay-card-balance_right(class="cursor-pointer" @click="toggleBalance")
                v-icon(v-if="isBalanceVisible") $eye
                v-icon(v-else).eye-off mdi-eye-off
            .mpay-card-date
              .mpay-card-date_number(:class="[!isBalanceVisible ? 'blur-balance': '']")
                span.mpay-card-date_text {{ getMaskedCard(mpayCobrandCard?.cardNumber) }}
                //- v-icon(@click="copyCardNumber(mpayCobrandCard?.cardNumber)") $copy
              span.mpay-card-date_text(:class="[!isBalanceVisible ? 'blur-balance': '']") {{ $moment(mpayCobrandCard?.cardExp).format('MM/YY')}}
            span.blockBlurContent(v-if="isLock") {{ $t('label.blocked')}}
          ul.mpay-card_action
            li.mpay-card_action_item
              v-icon.mpay-card_action_item_icon.v-btn--disabled(@click="$router.push({name: 'cobrand-topup'})") $plus
              span.mpay-card_action_item_text {{ $t('action.top_up.balance' )}}
            li.mpay-card_action_item.v-btn--disabled(@click="$router.push({ name: 'cobrand-transfers'})")
              v-icon.mpay-card_action_item_icon $send
              span.mpay-card_action_item_text {{ $t('action.send.money' )}}
            li.mpay-card_action_item(@click="lock")
              v-icon.mpay-card_action_item_icon $slash-circle
              span.mpay-card_action_item_text(v-if="isLock") {{ $t('action.unblock') }}
              span.mpay-card_action_item_text(v-else) {{ $t('action.block') }}
            li.mpay-card_action_item(@click="toggleCobrandSettings")
              v-icon.mpay-card_action_item_icon mdi-dots-horizontal
              span.mpay-card_action_item_text {{ $t('action.more' )}}
      CardHistory(v-if="mpayCobrandCard !== null")
      .cobrand-settings(v-if="isCobrandSettings" v-click-outside="onClickOutside")
        .cobrand-settings_header
          h3.cobrand-settings_header_title {{ $t('label.cobrand.settings.title') }}
          v-icon.cobrand-settings_header_icon(@click="toggleCobrandSettings") $close
        ul.cobrand-settings_menu
          li.cobrand-settings_menu_list(v-for="(item,index) in cardMenuList" @click="openCardDialog(index)" :class = "item.disabled ? 'itemDisabled' : ''")
            v-icon.cobrand-settings_menu_list_icon.icon-bg-blue {{item.icon}}
            span.cobrand-settings_menu_list_text {{item.title}}
            template(v-if="index === 6")
              v-icon(v-if="isDeposit") $deposit-off
              v-icon(v-else) $deposit-on
    .tabs__content(:class='{"isActive": (1 == selectedIndex)}')
      SavedCards(:cards="cards" :isCobrandCard='isCobrandCard')
  component(
    :is="currentForm"
    :is-modal-visible.sync='isModalVisible'
    :cards-list.sync="cards"
    :card-label.sync="label"
    :card="opCard"
    @clicked-islock="clickedIslock")
</template>
<script>
import AppLoader from '@/components/AppLoader';
import { PaymentRequests } from '@/services/http/payment-requests';
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import AppDialogModal from '@/components/AppDialogModal';
import CardHistory from '@/views/dashboard/pages/cards/components/card-history';
import SavedCards from '@/views/dashboard/pages/cards/components/saved-cards';
import CardInfo from '@/views/dashboard/pages/cards/modal/card-info';
import CardRekvizit from '@/views/dashboard/pages/cards/modal/card-rekvizit';
import CardPinChange from '@/views/dashboard/pages/cards/modal/card-pin-change';
import CardBlock from '@/views/dashboard/pages/cards/modal/card-block';
import CardUnblock from '@/views/dashboard/pages/cards/modal/card-unblock';
import DepositActive from '@/views/dashboard/pages/cards/modal/deposit-active';
import DepositDeactive from '@/views/dashboard/pages/cards/modal/deposit-deactive';
import CobrandNameUpdate from '@/views/dashboard/pages/cards/modal/name-update';
import AppGoBackIcon from '@/components/AppGoBackIcon';

export default {
  name: 'CobrandIndexPage',
  components: {
    AppLoader,
    AppGoBackIcon,
    AppDialogModal,
    CardHistory,
    CardInfo,
    CardRekvizit,
    CardPinChange,
    CardBlock,
    CardUnblock,
    SavedCards,
    CobrandNameUpdate,
    DepositActive,
    DepositDeactive,
  },
  data() {
    return {
      isDeposit: false,
      isBalanceVisible: true,
      isSavedCard: false,
      isCobrandCard: false,
      isLock: false,
      isLoading: true,
      isCobrandSettings: false,
      cardOrderInfo: null,
      isCardOrderInfo: false,
      opCard: {},
      label: '',
      currentForm: '',
      isModalVisible: false,
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
      cardMenuList: [
        { icon: '$blue-info', title: this.$t('label.info.popup.title'), disabled: false },
        { icon: '$edit-card-active', title: this.$t('action.rename'), disabled: false },
        { icon: '$blue-file', title: this.$t('action.account.requisites'), disabled: false },
        { icon: '$lock-blue', title: this.$t('action.change.pin'), disabled: false },
        { icon: '$shield-off', title: this.$t('action.reset.pin'), disabled: true },
        { icon: '$line-chart-down', title: this.$t('action.set.limits'), disabled: true },
        { icon: '$percent', title: this.$t('action.deposit.interest'), disabled: true },
      ],
      selectedIndex: 0,
      tabs: [
        {
          title: 'label.my.mpay.card',
        },
        {
          title: 'label.saved.cards',
        },
      ],
      mpayCobrandCard: null,
    };
  },
  async mounted() {
    await MpayCobrandRequests.cardOrderList().then((response) => {
      if (response[0].data !== null) {
        this.cardOrderInfo = response[0].data;
        this.isCardOrderInfo = this.cardOrderInfo.status === 'IN_PROGRESS';
      }
    }).catch((error) => {
      console.log(error);
    }).finally(() => {});

    const [response] = await PaymentRequests.savedCards();

    if (response) {
      this.cards = response?.data;
      const isCardholderName = this.cards.find((card) => { return card.cardholderName !== null; });
      if (isCardholderName || this.cardOrderInfo !== null) {
        this.selectTab(0);
        this.getMpayCobrandCard();
        if (!isCardholderName && this.cards.length > 0) {
          this.isLoading = false;
          this.isSavedCard = true;
        }
      } else {
        this.$router.push({ name: 'cobrand-info' });
      }
    }
  },
  methods: {
    selectTab(i) {
      this.selectedIndex = i;
    },
    async getMpayCobrandCard() {
      await MpayCobrandRequests.getMpayCobrandCard().then((response) => {
        this.mpayCobrandCard = response[0].data?.response[0];
        this.isLoading = false;
        if (this.mpayCobrandCard !== null) {
          this.cardMenuList[2].disabled = this.mpayCobrandCard.iban === null;
          this.isLock = this.mpayCobrandCard.isLock;
          this.isCobrandCard = true;
        } else {
          this.isSavedCard = true;
        }
      }).catch(() => {
        this.isLoading = false;
        this.isSavedCard = true;
      });
    },
    getMaskedCard(cardNo) {
      if (cardNo) {
        // Extract the first 6 digits and the last 4 digits
        const firstSix = cardNo?.substring(0, 6);
        const lastFour = cardNo?.substring(cardNo.length - 4);

        // Mask the digits in between with '*'
        const maskedDigits = cardNo?.substring(6, cardNo.length - 4).replace(/[a-zA-Z0-9]/g, '*');
        // Concatenate the masked parts
        const text = `${firstSix}${maskedDigits}${lastFour}`;
        return `${text.slice(0, 4)} ${text.slice(4, 8)} ${text.slice(8, 12)} ${text.slice(12, 16)}`;
      }

      return '';

    },
    toggleBalance() {
      this.isBalanceVisible = !this.isBalanceVisible;
    },
    lock() {
      this.opCard = {
        cardId: this.mpayCobrandCard?.cardId,
      };
      if (this.isLock) {
        this.currentForm = 'CardUnblock';
      } else {
        this.currentForm = 'CardBlock';
      }
      this.isModalVisible = true;
    },
    toggleCobrandSettings() {
      const overlay = document.getElementById('inspire');
      if (!this.isCobrandSettings) {
        overlay.classList.add('isCobrandSettings');
        this.isCobrandSettings = true;
      } else {
        overlay.classList.remove('isCobrandSettings');
        this.isCobrandSettings = false;
      }
    },
    async openCardDialog(index) {
      this.opCard = {};
      if (index === 0) {
        this.opCard = {
          cardHolder: this.mpayCobrandCard?.clientName,
          cardNumber: this.getMaskedCard(this.mpayCobrandCard?.cardNumber),
          expiryDate: this.$moment(this.mpayCobrandCard?.cardExp).format('MM/YY'),
        };
        this.currentForm = 'CardInfo';
      } else if (index === 1) {
        this.opCard = this.mpayCobrandCard;
        this.currentForm = 'CobrandNameUpdate';
      } else if (index === 2) {
        this.opCard = {
          iban: this.mpayCobrandCard?.iban,
          userId: this.mpayCobrandCard?.userId,
        };
        this.currentForm = 'CardRekvizit';
      } else if (index === 3) {
        this.currentForm = 'CardPinChange';
        this.opCard = {
          cardId: this.mpayCobrandCard?.cardId,
        };
      } else {
        this.currentForm = '';
      }
      this.isModalVisible = true;
    },
    copyCardNumber(cardNumber) {
      // this peace of code doesnt work on dev mode cause dev mod is in http without ssl , cause of that dev mode does not have ssl it works only with https
      navigator.clipboard.writeText(cardNumber);
      this.$notify.success(this.$t('misc.receipt.code.copy'));
    },
    clickedIslock(value) {
      this.isLock = value;
    },
    toggleIsDeposit() {
      if (this.isDeposit) {
        this.currentForm = 'DepositActive';
        this.isModalVisible = true;
      } else {
        this.currentForm = 'DepositDeactive';
        this.isModalVisible = true;
      }
      this.isDeposit = !this.isDeposit;
    },
    onClickOutside() {
      this.toggleCobrandSettings();
    },
  },
};
</script>

<style scoped lang="scss">
@import '@/styles/views/cards.scss';
@import '@/styles/components/empty.scss';
.v-btn--disabled{
  opacity: 0.5;
}

.tabs__content{
  display: none;
  width: 100%;
  &.isActive{
    display: block;
  }
}
.tabs{
  display: flex;
  gap: 12px;
  background: transparent;
  padding: 0;
  margin: 16px 0 24px;
  width: 100%;
  &-item{
    padding: 0;
    border-radius: 6px;
    min-height: 37px;
    color: #4B4B4B !important;
    text-align: center;
    font-size: 14px;
    font-weight: 500;
    line-height: 120%; /* 16.8px */
    flex: none;
    padding: 0 16px;
    border: 1px solid #E0E1E4;
    background: #FFF;
    cursor: pointer;
    &.tabs__selected{
      background: #06BDE9;
      color: #FFF !important;
      border:none;
    }
  }
}
.card-order_left{
  padding-left: 32px;
}
.card-saved{
  padding-bottom: 8px;
}
.card-saved-info_left{
  width: 55%;
}
.cobrand-settings{
  position: fixed;
  top: 0px;
  right: 0;
  z-index: 5;
  width: 464px;
  height: 100%;
  background: #FAFAFA;
  box-shadow: -5px 8px 15px 0px rgba(6, 6, 6, 0.10);
  padding: 50px 32px;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  &_header{
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    &_title{
      color: #262626;
      font-size: 20px;
      font-weight: 600;
    }
    &_icon{
      width: 14px;
      height: 14px;
      cursor: pointer;
    }
  }
  &_menu{
    padding: 0;
    display: flex;
    flex-direction: column;
    max-width: 400px;
    width: 100%;
    border-radius: 8px;
    border: 1px solid #E0E1E4;
    background: #FFF;
    margin-top: 24px;
    &_list{
      display: flex;
      list-style-type: none;
      border-bottom: 1px solid #ECECEC;
      padding: 16px 32px 10px;
      gap: 20px;
      align-items: center;
      cursor: pointer;
      &.itemDisabled {
        background: rgba(128, 128, 128, 0.1098039216);
        opacity: 0.5;
        pointer-events: none;
      }
      &:hover{
        background: #8080801c;
      }
      &:first-of-type{
        padding-top: 32px;
      }
      &:last-of-type{
        border-bottom: none;
        padding-bottom: 32px;
      }
      &_icon{
        width: 44px;
        height: 44px;
      }
      &_text{
        color:  #4B4B4B;
        font-size: 16px;
        font-weight: 500;
        line-height: 150%; /* 24px */
        white-space: nowrap;
      }
    }
  }
}
.card-action{
  display: flex;
  align-items: flex-end;
  position: absolute;
  bottom: 50px;
  left: 50px;
  gap: 19px;
  justify-content: space-between;
  width: calc(100% - 50px);
}
.card-rectangle{
  position: relative;
  .card-img {
    background: url('@/assets/rectangle-card.svg');
    height: 286px;
    border-radius: 20px 20px 0px 0px;
    background-size: cover;
    &::after{
      content: "";
      height: 286px;
      width: 100%;
      position: absolute;
      top: 0;
      left: 0;
      background: rgba(3, 79, 98, 0.60);
      border-radius: 20px 20px 0px 0px;
    }
  }
  .extract{
    border-radius: 10px;
    background: #36C135;
    position: absolute;
    top: 24px;
    right: 24px;
    padding: 11px 16px;
    display: flex;
    gap: 10px;
    pointer-events: none;
    opacity: 0.5;
    &_text{
      color: #FFF;
      font-size: 12px;
      font-weight: 500;
      line-height: 14px;
    }
    &_icon{
      width: 14px;
      height: 14px;
    }
  }
}
.mpay-card{
  background: url('@/assets/mpay-card.svg') lightgray 50% / cover no-repeat;
  border-radius: 20px;
  background: linear-gradient(270deg, );
  padding: 22px 24px;
  display: flex;
  flex-direction: column;
  width: 336px;
  height: 186px;
  position: relative;
  justify-content: space-between;
  .blockBlurContent {
    position: absolute;
    transform: translate(-50%, -50%);
    z-index: 1;
    top: 50%;
    left: 50%;
     color: white;
    font-size: 12px;
    font-weight: 500;
  }
  &.isBlockBlur::after{
    content: "";
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.70);
    backdrop-filter: blur(1.5px);
    position: absolute;
    top: 0;
    left: 0;
    border-radius: 20px;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  &-visa{
    display: flex;
    gap: 52px;
    justify-content: space-between;
    &_left{
      display: flex;
      gap: 9px;
      align-items: center;
      &_icon{
        width: 40px;
        height: 40px;
        border-radius: 50%;
        padding: 8px;
        border: 1px solid #8CE1F5;
        :deep(.v-icon__component){
          width: 100%;
          path{
            stroke: white;
          }
        }
      }
      &_text{
        color: white;
        font-size: 14px;
        font-weight: 500;
        line-height: 120%; /* 16.8px */
      }
    }
    &_right{
      width: 40px;
      :deep(.v-icon__component){
        width: 100%;
      }
    }
  }
  &_clientName{
    color: white;
    font-size: 14px;
    margin: 10px 0;
    font-weight: 500;
  }
  &-balance{
    display: flex;
    justify-content: space-between;
    align-items: center;
    &_left{
      font-size: 22px;
      font-weight: 500;
      display: flex;
      gap: 8px;
      color: white;
      align-items: baseline;
      span:last-of-type {
        font-size: 20px;
        font-weight: 400;
      }
    }
    &_right{
      cursor: pointer;
      .eye-off{
        color: white;
      }
    }
  }
  &-date{
    border-top: 1px solid #B2EBF8;
    padding-top: 12px;
    display: flex;
    justify-content: space-between;
    &_number{
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 16px;
      :deep(.v-icon__component){
        path {
          stroke:white;
        }
      }
    }
    &_text{
      color: #E6F8FD;
      font-size: 14px;
      font-weight: 500;
      line-height: 17px;
      padding: 3.5px 0;
    }
  }
  &_action{
    padding: 0;
    display: flex;
    gap: 10px;
    &_item{
      list-style-type: none;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 10px;
      width: 75px;
      justify-content: center;
      cursor: pointer;
      &_icon{
        display: flex;
        width: 48px;
        min-height: 48px;
        padding: 8px;
        justify-content: center;
        align-items: center;
        border-radius: 10px;
        background: rgba(255, 255, 255, 0.70);
        box-shadow: 0px 4.817px 8.029px 0px rgba(0, 0, 0, 0.10);
        :deep(.v-icon__component){
          path{
            stroke: #262626;
          }
        }
        .mdi.mdi-dots-horizontal{
          color: #262626;
        }
        &:hover{
          background: white;
        }
      }
      &_text{
        color: #FFF;
        font-size: 12px;
        font-weight: 500;
        line-height: 120%; /* 14.4px */
        max-height: 30px;
        height: 100%;
        text-align: center;
      }
    }
  }
}
@media (max-width: 650px) {
  .card-saved-list {
    border-radius: 10px;
    padding: 20px 16px;
    margin-top: 0;
    &_item{
      padding: 16px;
    }
  }
  .card-action{
    flex-direction: column;
    align-items: center;
    justify-content: center;
    bottom: 32px;
    gap: 27px;
    left: 0;
    width: calc(100% - 0px);
  }
  .cobrand-settings {
    width: 100%;
    justify-content: start;
    padding: 44px 16px 32px;
    height: 100vh;
  }
  .card-rectangle {
    .card-img {
      height: 349px;
      &::after{
        height: 349px;
      }
    }
    .extract{
      display: none;
    }
  }
  .tabs{
    gap: 8px;
    margin: 24px 16px 20px;
    padding: 0 16px;
    &-item{
      padding: 0 12px;
    }
  }
  .mpay-card {
    width: 343px;
  }
}
</style>
