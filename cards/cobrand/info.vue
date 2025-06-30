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
    .card-order
      .card-saved-info
        .card-saved-info_left
          h1.card-saved-info_left_title {{$t('label.card.order.detail.title')}}
          p.card-saved-info_left_desc {{$t('label.card.order.detail.desc')}}
          .card-saved-info_left_buttons
            v-btn.card-saved-info_left_button(v-if="getUserData?.profile?.extraParams?.enable_card_order" @click="scroll('cardOrderForm', 3)") {{$t('action.card.order')}}
            span(v-else).soon-btn {{ $t('action.soon') }}
        .card-saved-info_right
          img(src="@/assets/card-detail.png")
      .scroll-tabs
        v-list
          v-list-item(@click="scroll('tabcontent0',0)" class="tablinks active") {{$t('label.card.tabLink1')}}
          v-list-item(@click="scroll('tabcontent1',1)" class="tablinks")  {{$t('label.card.tabLink2')}}
          v-list-item(@click="scroll('tabcontent2',2)" class="tablinks") {{$t('label.card.tabLink3')}}
        .about#tabcontent0
          .about-item(v-for="card in about" :style="card.bgColor ? `background-color: ${card.bgColor}` : ''")
            .about-item-img
              img(:src="require('@/assets/' + card.name)")
            .about-item-title {{ $t(card.title) }}
            .about-item-text {{ $t(card.desc) }}
        .advantages#tabcontent1
          h2.card-title {{ $t('label.card.benefits') }}
          .advantages-container
            .advantages-container-item(v-for="advantage in advantages")
              img.advantages-container-item_img(:src="require('@/assets/' + advantage.name)")
              h3.advantages-container-item_title {{ $t(advantage.title) }}
              p.advantages-container-item_desc {{ $t(advantage.desc) }}
        .additional#tabcontent2(v-if="getUserData?.profile?.extraParams?.enable_card_order")
          v-expansion-panels.additional-panels
            v-expansion-panel.additional-panels-item(v-for="(panel,index) in panels" :key="index")
              v-expansion-panel-header(expand-icon="mdi-plus").additional-panels-item_header {{ $t(panel.header)}}
              v-expansion-panel-content.additional-panels-item_content {{ $t(panel.content) }}
          .additional-stepper
            h2.card-title.additional-stepper_title {{ $t('label.how.get.card') }}
            v-stepper.additional-stepper-container(alt-labels)
              v-stepper-header
                v-stepper-step.additional-stepper-container_item(step="1" complete)  {{ $t('label.fill.form') }}
                v-stepper-step.additional-stepper-container_item(step="2" complete) {{ $t('label.follow.notification') }}
                v-stepper-step.additional-stepper-container_item(step="3" complete) {{ $t('label.get.card') }}
          .card-order-form#cardOrderForm
            CardOrderForm

</template>

<script>
import CardOrderForm from '@/views/dashboard/pages/cards/components/card-order-form';
import AppGoBackIcon from '@/components/AppGoBackIcon';
import { mapGetters } from 'vuex';

export default {
  name: 'CobrandInfo',
  components: {
    AppGoBackIcon,
    CardOrderForm,
  },
  data() {
    return {
      panels: [
        { header: 'label.panels.header1', content: 'label.panels.content1' },
        { header: 'label.panels.header2', content: 'label.panels.content2' },
        { header: 'label.panels.header3', content: 'label.panels.content3' },
        { header: 'label.panels.header4', content: 'label.panels.content4' },
      ],
      about: [
        {
          name: 'tab11.svg', title: 'label.annual.service.fee', desc: 'label.free', bgColor: '#E6F8FD',
        },
        {
          name: 'tab12.svg', title: 'label.duration', desc: 'label.years', bgColor: '#FDEAEA',
        },
        {
          name: 'tab13.svg', title: 'label.currency', desc: 'AZN', bgColor: '#EBF9EB',
        },
        {
          name: 'tab14.svg', title: 'label.cashback', desc: 'label.percent', bgColor: '#FFF4ED',
        },
      ],
      advantages: [
        { name: 'tab21.svg', title: 'label.advantages.title1', desc: 'label.advantages.desc1' },
        { name: 'tab22.svg', title: 'label.advantages.title2', desc: 'label.advantages.desc2' },
        { name: 'tab23.svg', title: 'label.advantages.title3', desc: 'label.advantages.desc3' },
        { name: 'tab24.svg', title: 'label.advantages.title4', desc: 'label.advantages.desc4' },
        { name: 'tab25.svg', title: 'label.advantages.title5', desc: 'label.advantages.desc5' },
        { name: 'tab25.svg', title: 'label.advantages.title6', desc: 'label.advantages.desc6' },
      ],
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
      registerForm: true,
    };
  },
  computed: {
    ...mapGetters('user', ['getUserData']),
  },
  methods: {
    scroll(tabcontent, index) {
      const tablinks = document.getElementsByClassName('tablinks');
      for (let i = 0; i < tablinks.length; i++) {
        tablinks[i].classList.remove('active');
        if (index !== 3 && index === i) {
          tablinks[i].classList.add('active');
        }
      }
      const element = document.getElementById(tabcontent);
      window.scroll({ top: index !== 0 ? element.offsetTop + 30 : element.offsetTop, left: 0, behavior: 'smooth' });

    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/cards.scss';
.card-saved-info{
  padding: 0;
  gap:0;
  &_left {
    padding: 34px 0;
    &_title{
      width: 55%;
    }
  }
}
.additional-panels-item_header :deep(.v-icon){
  color:#06BDE9 !important;
}
.v-expansion-panel--active > .v-expansion-panel-header{
  font-weight: 600;
  padding-bottom: 10px;
}
:deep(.v-expansion-panel--active > .v-expansion-panel-header--active .v-expansion-panel-header__icon:not(.v-expansion-panel-header__icon--disable-rotate) .v-icon){
  transform: rotate(-180deg);
  &::before{
    content: "\F0374";
  }
}
:deep(.v-expansion-panel > .v-expansion-panel-header .v-expansion-panel-header__icon .v-expansion-panel-header__icon .v-icon) {
  transform: none;
  &::before{
    content: "\F0415";
  }
}
.scroll-tabs{
  .v-list{
    padding: 0;
    display: flex;
    gap: 40px;
    &-item{
      padding: 0;
      color: #7A7A7A !important;
      font-size: 16px;
      font-weight: 500;
      line-height: 150%; /* 24px */;
      min-height: 46px;
      flex: none;
      &:hover{
        color: #262626!important;
        &::before{
          opacity: 0;
        }
      }
      &.active{
        color: #262626!important;
      }
    }
  }
  .about{
    display: flex;
    gap: 16px;
    flex-wrap: wrap;
    justify-content: space-evenly;
    &-item{
      display: flex;
      padding: 24px 10px;
      flex-direction: column;
      align-items: center;
      gap: 6px;
      border-radius: 16px;
      background-color: white;
      justify-content: space-between;
      min-width: 117px;
      width: 100%;
      flex: 1;
      &-img{
        width: 118px;
        height: 100px;
        padding: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        img{
          width: 100%;
        }
      }
      &-title{
        color: #797D85;
        font-size: 14px;
        font-weight: 400;
        line-height: 125%; /* 17.5px */
        padding: 0;
        text-align: center;
      }
      &-text{
        padding: 0;
        text-align: center;
        color: #25282B;
        font-size: 16px;
        font-weight: 500;
        line-height: 150%; /* 24px */
        -moz-filter: blur(10px);
        -o-filter: blur(10px);
        -ms-filter: blur(10px);
        filter: blur(10px);
        -webkit-user-select: none;
        -khtml-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        -o-user-select: none;
        user-select: none;
      }
    }
  }
  .advantages{
    padding-top: 50px;
    &-title{
      color: #262626;
      text-align: center;
      font-size: 20px;
      font-weight: 600;
      line-height: 150%; /* 30px */
    }
    &-container{
      display: flex;
      grid-gap: 16px;
      padding-top: 32px;
      flex-wrap: wrap;
      justify-content: space-evenly;
      &-item{
        display: flex;
        flex-direction: column;
        padding: 24px;
        gap: 10px;
        border-radius: 16px;
        background-color: white;
        box-shadow: 0px 8px 30px 0px #E7E5E5;
        align-items: flex-start;
        max-width: 215px;
        width: 100%;
        &_img{
          height: 50px;
          width: auto;
          padding: 0;
          margin-bottom: 10px;
        }
        &_title{
          color: #262626;
          font-size: 16px;
          font-weight: 500;
          line-height: 24px;
          width: 169px;
          text-overflow: ellipsis;
          display: -webkit-box;
          -webkit-line-clamp: 2;
          -webkit-box-orient: vertical;
          overflow: hidden;
        }
        &_desc{
          text-align: left;
          color: #797D85;
          font-size: 14px;
          font-weight: 400;
          height: 125px;
          overflow: hidden;
          margin-bottom: 0;
          line-height: 175%; /* 24.5px */
          text-overflow: ellipsis;
          display: -webkit-box;
          -webkit-line-clamp: 5;
          -webkit-box-orient: vertical;
        }
      }
    }
  }
  .additional {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 50px;
    padding-top: 50px;
    &-panels{
      background: #FAFAFA;
      padding: 50px 32px;
      gap: 16px;
      &-item{
        min-width: 512px;
        border-radius: 8px !important;
        border: 1px solid #E0E1E4;
        &::before{
          box-shadow: none;
        }
        &::after{
          border: none;
        }
        &_header{
          color: #4B4B4B;
          font-size: 16px;
          font-weight: 400;
          line-height: 150%; /* 24px */
          padding: 15px 24px;
          min-height: 24px;
          .v-icon{
            color: #06bde9!important;
          }
        }
        &_content{
          color:  #7A7A7A;
          font-size: 14px;
          font-weight: 400;
          line-height: 175%; /* 24.5px */
          width: 90%;
        }
      }
    }
    &-stepper{
      width: 100%;
      background: white;
      max-width: 776px;
      width: 100%;
      &_title{
        margin-bottom: 24px;
      }
      &-container{
        box-shadow: none !important;
        &_item{
          flex-basis: auto;
          padding: 0 24px;
          :deep(.v-icon){
            display: none;
          }
          :deep(.v-stepper__label) {
          font-size: 14px;
          font-weight: 500;
          color: #4B4B4B !important;
          }
          :deep(.v-stepper__step__step.primary){
            position: relative;
            border-radius: 20px;
            border: 2px solid #FFF;
            background:#06BDE9;
            border-color: #fff !important;
            box-shadow: 0px 1px 1px 0px rgba(0, 0, 0, 0.20), 0px 2px 2px 0px rgba(0, 0, 0, 0.14), 0px 1px 5px 0px rgba(0, 0, 0, 0.12);
            &::before{
              content: "";
              position: absolute;
              top: 50%;
              left: -92px;
              background: #E0E1E4;
              width: 240px;
              transform: translate(50%, 50%);
              height: 1px;
            }
          }
        }
      }
    }
 }
 .additional-stepper-container_item:nth-child(2) :deep(.v-stepper__step__step.primary)::before{
    left: -90px;
    width: 234px;
  }
  .additional-stepper-container_item:nth-child(3) :deep(.v-stepper__step__step.primary)::before{
    width: 0;
  }
}
@media only screen and (max-width: 959.98px) {
  :deep(.v-stepper:not(.v-stepper--vertical) .v-stepper__label) {
    display: block;
  }
}
@media (max-width: 650px) {
  .wrapper{
    height: auto;
  }
  .v-text-field.v-text-field--enclosed.v-text-field--outlined.field{
      max-width: 100%;
  }
  .card-order {
    padding-top: 0;
    min-width: 100%;
  }
  .card-saved-info_left{
    padding-top: 32px;
    &_buttons{
      justify-content: space-between;
      gap: 16px;
    }
    &_title{
      width: 50%;
    }
  }
  .v-stepper--alt-labels .v-stepper__header{
     box-shadow: none;
     flex-wrap: nowrap;
     :deep(.v-stepper__label){
        text-align: center;
        padding: 0 10px;
        line-height: 17px;
     }
  }
  .scroll-tabs{
    .v-list{
      gap: 24px;
      &-item{
        padding: 0;
        color: #7A7A7A !important;
        font-size: 16px;
        font-weight: 500;
        line-height: 150%; /* 24px */;
        flex: none;
        &.active{
          color: #262626!important;
        }
      }
    }
    .about{
      &-item{
        min-width: 155px;
        flex: 0;
          &-img{
          width: 100px;
        }
      }
    }
    .advantages{
      padding-top: 32px;
      &-container{
        grid-gap: 16px;
        grid-auto-flow: row;
        grid-template-rows: 247px;
        &-item{
          max-width: 100%;
          height: auto;
          &_img{
            margin-bottom: 6px;
          }
          &_title{
            width: auto;
          }
          &_desc{
            height: 75px;
            padding-right: 20px;
            -webkit-line-clamp: 3;
          }
        }
      }
    }
    .additional {
      gap: 32px;
      padding-top: 32px;
      &-stepper {
        padding: 0px;
      }
      &-panels{
        padding: 32px 16px;
        :deep(.v-expansion-panel-header__icon) {
          padding-left: 24px;
        }
        &-item{
          min-width: 100%;
        }
      }
      .additional-stepper-container_item{
        padding: 0px;
        width: 22%;
      }
      .additional-stepper-container_item :deep(.v-stepper__step__step.primary)::before{
        left: -18px;
        width: 92px;
      }
      .additional-stepper-container_item:nth-child(2){
        width: 100%;
        max-width: 145px;
        :deep(.v-stepper__step__step.primary)::before{
          left: -17px;
          width: 90px;
        }
      }
      .additional-stepper-container_item:nth-child(3) :deep(.v-stepper__step__step.primary)::before{
        width: 0px;
      }
   }
  }
}
</style>
