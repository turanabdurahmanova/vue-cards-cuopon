<template lang="pug">
AppSheetComponent
  AppBreadCrumbs(
    pageName="label.coupons"
    :crumbs="items"
  )
  template(v-if="!couponLoading")
    v-row.mb-6(v-if="$vuetify.breakpoint.mobile" align="center")
      v-col
        AppGoBackIcon
      v-col.text-center
        div.text-h6.py-0 {{$t('label.coupons')}}
      v-col.d-flex.justify-end
    .empty-page(v-if="!getCouponList.length && !isFiltered")
      .empty-page_wrapp
        v-icon.empty-page_img $coupon-empty
        span.empty-page_txt {{$t('misc.coupon.index.empty')}}
        v-btn.empty-page_btn(id="transactions_go_dashboard" to="/" depressed) {{ $t('action.go_dashboard') }}
    template(v-else)
      .filter-form
        app-date-picker.filter-form_picker(v-model="date"
          :placeholder="$t('filters.date')"
          format="DD.MM.YYYY"
          :is-clearable="false"
          :range-presets="presets"
          no-header range validate)
        v-select.filter-form_select(:items="filterItems"
          :item-text="item => $t(item.text)"
          multiple
          v-model="filtering"
          :placeholder="$t('filters.coupons.types')"
          :menu-props="{offsetY: true}"
          hide-details
          outlined)
        v-btn.filter-form_btn(id="transactions_filter" fab depressed @click="submitFilter")
          v-icon(height="20px" width="20px") mdi-magnify
      .empty-page.isFiltered(v-if="isFiltered && !getCouponList.length")
        .empty-page_wrapp
          v-icon.empty-page_img $coupon-empty
          span.empty-page_txt {{ $t('misc.coupon.search.empty') }}
      template(v-else)
        v-list.customVList(subheader dense two-line v-for='(couponItem,index) in getCouponList')
          v-subheader.subheader
            span {{ couponItem[0].createdAt | dateFormat(couponItem[0].createdAt)}}
          template(v-for="(coupon, index) in couponItem")
            v-list-item(:key="index" @click="goToReceipt(coupon)")
              .title__icons
                v-icon {{getIcon(coupon)}}
                v-icon {{ getCouponStatus(coupon).icon }}
              v-list-item-content.py-0
                v-list-item-title {{ coupon.sender === false ? getTitle(coupon) + " 🥳" :  getTitle(coupon) }}
                v-list-item-subtitle(:class = "[{'red--text': getCouponStatus(coupon).subTitle !== coupon.phoneNumber}]") {{getCouponStatus(coupon).subTitle}}
              v-list-item-action-text
                span(:class="(coupon.status ==='ACCEPTED' || coupon.status ==='READY') && coupon.sender===true ? 'black--text' : 'green--text'") {{ getAmount(coupon) }} AZN
                span {{ $moment(coupon.createdAt).format('HH:mm, D.MM.YYYY') }}
              v-list-item-action
                v-menu(left offset-y)
                  template(v-slot:activator="{ on }")
                    v-icon(v-on="on") mdi-dots-vertical
                  v-list(style="min-width:190px").pa-2.px-0
                    v-list-item(v-for="option in coupon.operationOptions" @click.stop="option.onClick(coupon)"
                    :key="option.name" :id="option.gtmId !==null ? option.gtmId : ''").d-flex.align-center.px-5
                      v-list-item-icon.my-0.mr-3.align-self-center
                        v-progress-circular(:width="3" :size="30" color="primary" v-if="isDownloadReceipt && option.id===5" indeterminate)
                        v-icon(v-else v-text='option.icon')
                      v-list-item-title.menu-list-item-text {{ option.title }}
            v-divider
        AppPagination(:totalPages='totalPages' :page.sync='currentPage' @pagechanged="changePage")
    .empty-page(v-else)
      app-loader
</template>

<script>
import AppBreadCrumbs from '@/components/AppBreadCrumbs';
import { CouponRequests } from '@/services/http/coupon-requests';
import { sortRecordsByDate } from '@/helpers/sortRecordsByDate';
import { i18n } from '@/i18n';
import moment from 'moment';
import AppLoader from '@/components/AppLoader';
import { PaymentRequests } from '@/services/http/payment-requests';
import AppDatePicker from '@/components/AppDatePicker';
import AppSheetComponent from '@/components/AppSheetComponent';
import AppGoBackIcon from '@/components/AppGoBackIcon';
import AppPagination from '@/components/AppPagination';

const Variables = {
  DateFormat: 'DD.MM.YYYY',
};

export default {
  name: 'CouponsIndexPage',
  components: {
    AppBreadCrumbs,
    AppDatePicker,
    AppLoader,
    AppSheetComponent,
    AppPagination,
    AppGoBackIcon,
  },
  filters: {
    dateFormat(rawDate) {
      const today = moment();
      const yesterday = moment().subtract(1, 'day');

      if (moment(rawDate).isSame(today, 'day')) {
        return i18n.t('misc.history.day.today');
      }
      if (moment(rawDate).isSame(yesterday, 'day')) {
        return i18n.t('misc.history.day.yesterday');
      }
      return moment(rawDate).format(Variables.DateFormat);

    },
  },
  data() {
    return {
      currentPage: 1,
      items: [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          href: '',
        },
        {
          text: this.$t('label.coupons'),
          disabled: true,
          href: '',
        },
      ],
      filterItems: [
        {
          value: 'received',
          text: 'filters.coupon.types.received',
        },
        {
          value: 'sender',
          text: 'filters.coupon.types.sent',
        },
      ],
      filtering: [],
      date: {
        start: null,
        end: null,
      },
      couponLoading: true,
      isFiltered: false,
      isDownloadReceipt: false,
      sender: null,
      coupons: [],
      totalPages: 0,
    };
  },
  computed: {
    getCouponList() {
      const def = [
        {
          id: 1,
          title: this.$t('action.look.receipt'),
          icon: '$show-password-active',
          onClick: this.goToReceipt,
          gtmId: 'go_to_receipt',
        },
        {
          id: 5,
          title: this.$t('action.download'),
          icon: '$download-file',
          onClick: this.downloadReceipt,
          gtmId: 'download_receipt',
        },
      ];
      if (!this.coupons) {
        return [];
      }

      this.coupons.find((item) => {
        item.operationOptions = [...def];
        if ((item.sender && (item.status === 'REVERSE')) || (!item.sender && item.status === 'READY')) {
          item.operationOptions.push({
            id: 2,
            title: this.$t('action.transfer.wallet'),
            icon: '$coupon-wallet',
            onClick: this.topUp,
            gtmId: 'go_to_wallet',
          });
        }
        return item;
      });
      return sortRecordsByDate(this.coupons, 'createdAt');
    },
    filterParams() {
      const filterObj = {
        sender: null,
      };
      if (this.filtering.length === 0 || this.filtering.length === 2) {
        return '';
      }
      return this.filtering.reduce((a, b) => {
        if (b === 'sender') {
          a.sender = true;
        } else {
          a.sender = false;
        }
        return a;
      }, filterObj);
    },
    dateParams() {
      return {
        fromDate: this.date?.start,
        toDate: this.date?.end,
      };
    },
    presets() {
      return [
        {
          name: this.$t('filters.period.this_week'),
          dates: {
            start: this.$moment().startOf('isoWeek').toString(),
            end: new Date(),
          },
        },
        {
          name: this.$t('filters.period.last_month'),
          dates: {
            start: this.$moment().subtract(1, 'months').startOf('day').toString(),
            end: new Date(),
          },
        },
        {
          name: this.$t('filters.period.last_months', { number: 3 }),
          dates: {
            start: this.$moment().subtract(3, 'months').startOf('day').toString(),
            end: new Date(),
          },
        },
      ];
    },
    breadcrumbs() {
      return [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          exact: true,
          to: {
            name: 'dashboard-root',
          },
        },
      ];
    },
  },
  async mounted() {
    this.couponLoading = true;
    const [coupons] = await CouponRequests.getCoupons({
      params: {
        page: 1,
        size: 7,
      },
    });
    this.totalPages = coupons?.data?.totalPages;
    this.coupons = coupons?.data?.list;
    this.couponLoading = false;
  },
  methods: {
    async changePage() {
      this.couponLoading = true;
      let params = {
        size: 7,
        page: this.currentPage,
      };
      const changePageParam = { ...this.filterParams, ...this.dateParams };
      this.isFiltered = Object.values(changePageParam).some((value) => value !== null);
      if (this.isFiltered) {
        params = {
          size: 7,
          page: this.currentPage,
          fromDate: changePageParam.fromDate,
          toDate: changePageParam.toDate,
          sender: changePageParam.sender,
        };
      }
      this.coupons = await CouponRequests.getCoupons({
        params,
      }).then((result) => result[0]?.data?.list);
      this.couponLoading = false;
    },
    submitFilter() {
      this.page = 1;
      this.changePage();
    },
    goToReceipt(item) {
      this.$router.push({
        name: 'coupon-receipt',
        params: { id: item.couponId },
      });
    },
    async topUp(item) {
      const [response, error] = await PaymentRequests.topUp({
        couponId: item.couponId,
        paymentChannel: 'COUPON',
        amount: item.amount,
      });
      console.log(response);
      if (error) {
        this.$notify.error(error?.errorDetails[0].message);
      } else {
        this.$notify.success(this.$t('misc.notify.general.success'));
      }
    },
    getTitle(item) {
      if (item.sender) {
        return this.$t('label.coupon.sent');
      }
      return this.$t('label.coupon.coming');
    },
    getIcon(item) {
      if (item.sender) {
        return '$arrow-up-right';
      }
      return '$arrow-down-left';
    },
    getCouponStatus(coupon) {
      if (coupon.sender) {
        switch (coupon?.status) {
          case 'PENDING':
            return {
              subTitle: coupon.phoneNumber,
              icon: '$coupon-info-circle',
            };
          case 'READY':
            return {
              subTitle: coupon.phoneNumber,
              icon: '$coupon-info-circle',
            };
          case 'ACCEPTED':
            return {
              subTitle: coupon.phoneNumber,
              icon: '$coupon-check-circle',
            };
          case 'REJECTED':
            return {
              subTitle: 'Qəbul edilmədi',
              icon: '$coupon-check-circle',
              subTitleClass: 'red',
            };
          default:
            return {
              subTitle: 'Qəbul edilmədi',
              icon: '$coupon-info-circle',
              subTitleClass: 'red',
            };
        }
      } else {
        switch (coupon?.status) {
          case 'READY':
            return {
              subTitle: coupon.phoneNumber,
              icon: '$coupon-info-circle',
            };
          case 'ACCEPTED':
            return {
              subTitle: coupon.phoneNumber,
              icon: '$coupon-check-circle',
            };
          default:
            return '';
        }
      }
    },
    getAmount(item) {
      if (item.sender) {
        if (item.status === 'REJECTED' || item.status === 'REVERSE') return `+${item.amount.toFixed(2)}`;
        return `-${item.amount.toFixed(2)}`;
      }
      return `+${item.amount.toFixed(2)}`;
    },
    async downloadReceipt(item) {
      this.isDownloadReceipt = true;
      const req = {
        couponCode: item.couponId,
      };
      await CouponRequests.downloadReceipt(req, { responseType: 'blob' }).then((response) => {
        const url = window.URL.createObjectURL(response[0]);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'coupon';
        a.click();
        a.remove();
        setTimeout(() => window.URL.revokeObjectURL(url), 100);
      }).catch(() => {
        this.$notify.error(this.$t('server.error.internal'));
        this.isDownloadReceipt = false;
      });
      this.isDownloadReceipt = false;
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/views/cards.scss';
@import '@/styles/components/empty.scss';
@import '@/styles/components/filter-form.scss';

.title__icons{
  box-shadow: 0px 2px 10px 0px rgba(0, 0, 0, 0.16);
  width: 48px;
  height: 48px;
  background: white;
  border-radius: 50%;
  margin-right: 16px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  .v-icon:last-child {
    position: absolute;
    top: 0;
    right: 0;
    :deep(.v-icon__component) {
      height: 13px !important;
      width: 13px !important;
    }
  }
}

</style>
