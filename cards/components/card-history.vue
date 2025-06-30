<template lang="pug">
  .loader(v-if="isLoading && !isFiltered")
    app-loader
  AppSheetComponent(v-else)
    template(v-if="mpayCobrandCards?.length>0 || mpayCobrandCards!==null")
      h2.list-title {{ $t('label.cobrand.history.title') }}
      .filter-form
        app-date-picker.filter-form_picker(v-model="date"
          :placeholder="$t('filters.date')"
          format="DD.MM.YYYY"
          :is-clearable="false"
          :range-presets="presets"
          no-header range validate)
        v-btn.filter-form_btn(id="transactions_filter" fab depressed @click="submitFilter")
          v-progress-circular(v-if="isLoading" :width="3" :size="30" color="primary" indeterminate)
          v-icon(v-else height="20px" width="20px") mdi-magnify
      v-list.customVList(subheader dense two-line v-for='(records,index) in getRecords')
        v-subheader
          span {{ records.createdAt | dateFormat(records.createdAt)}}
        template(v-for="(record, index) in records")
          v-list-item(:key="index" @click="goToReceipt(record)")
            v-list-item-avatar.my-0
              img(src="https://logo-mpay.s3.amazonaws.com/rabitabank.svg")
            v-list-item-content.py-0
              v-list-item-title(v-text="record.description")
              v-list-item-subtitle.caption.text--disabled(v-text="record.contractNumber")
            v-list-item-action-text
              span(:class = "getAmount(record) > 0 ? 'green--text' : 'black--text'") {{ getAmount(record) }} AZN
              span {{ $moment(record.createdAt).format('HH:mm, D.MM.YYYY') }}
            v-list-item-action
              v-menu(left offset-y)
                template(v-slot:activator="{ on }")
                  v-icon(v-on="on") mdi-dots-vertical
                v-list(style="min-width:190px").pa-2.px-0
                  v-list-item(v-for="option in record.operationOptions" @click.stop="option.onClick(record)"
                  :key="option.name" :id="option.gtmId !==null ? option.gtmId : ''").d-flex.align-center.px-5
                    v-list-item-icon.my-0.mr-3.align-self-center
                      v-icon(v-text='option.icon')
                    v-list-item-title.menu-list-item-text {{ option.title }}
          v-divider
        AppPagination(v-if="mpayCobrandCards?.length>4" :totalPages='getPages' :page.sync='currentPage' @pagechanged="updatePage")
    .empty-page(v-else)
      .empty-page_wrapp
        v-img.empty-page_img(src="@/assets/empty-card.png")
        span.empty-page_txt {{ $t('misc.history.index.empty') }}
        v-btn.empty-page_btn(id="cards_go_dashboard_root" :to="{name: 'dashboard-root'}" depressed) {{ $t('action.go_dashboard') }}
</template>
<script>
import { MpayCobrandRequests } from '@/services/http/mpay-cobrand-requests';
import { sortRecordsByDate } from '@/helpers/sortRecordsByDate';
import AppDatePicker from '@/components/AppDatePicker';
import AppSheetComponent from '@/components/AppSheetComponent';
import moment from 'moment';
import { i18n } from '@/i18n';
import AppLoader from '@/components/AppLoader';
import AppPagination from '@/components/AppPagination';

const Variables = {
  DateFormat: 'DD.MM.YYYY',
};

export default {
  name: 'TransactionsIndexPage',
  components: {
    AppLoader,
    AppDatePicker,
    AppSheetComponent,
    AppPagination,
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
      filterItems: [
        {
          value: 'TOP_UP',
          text: 'Kommunal',
        },
        {
          value: 'TRANSFER',
          text: 'Mobil',
        },
        {
          value: 'SAVED_CARD',
          text: 'Bank',
        },
        {
          value: 'WALLET',
          text: 'Dövlət və bələdiyyə',
        },
        {
          value: '1',
          text: 'TV',
        },
        {
          value: '2',
          text: 'İnternet',
        },
        {
          value: '3',
          text: 'Telefon',
        },
      ],
      filtering: [],
      currentPage: 1,
      perPage: 4,
      mpayCobrandCards: [],
      items: [
        {
          text: this.$t('label.dashboard'),
          disabled: false,
          href: '',
        },
        {
          text: this.$t('label.history'),
          disabled: true,
          href: '',
        },
      ],
      date: {
        end: null,
        start: null,
      },
      isFiltered: false,
      errorCases: ['CHARGE_ERROR', 'PAYMENT_FAILED', 'PRE_CHARGE_EXCEPTION', 'REVERSE_INPROGRESS', 'REVERSED', 'ERROR', 'AUTO_REVERSE', 'EXPIRED'],
      incomeCases: ['TRANSFER_TARGET', 'TOP_UP'],
      isLoading: false,
    };
  },
  computed: {
    getRecords() {
      const def = [
        {
          id: 1,
          title: 'Qəbzə bax',
          icon: '$show-password-active',
          onClick: this.goToReceipt,
          gtmId: 'go_to_receipt',
        },
        {
          id: 5,
          title: 'Yüklə',
          icon: '$download-file',
          onClick: this.downloadReceipt,
          gtmId: 'download_receipt',
        },
      ];
      if (!this.mpayCobrandCards) {
        return [];
      }
      const startIndex = this.perPage * (this.currentPage - 1);
      const endIndex = startIndex + this.perPage;
      const displayedRecords = this.mpayCobrandCards.slice(startIndex, endIndex);
      displayedRecords.find((item) => {
        item.operationOptions = [...def];
        return item;
      });
      return sortRecordsByDate(displayedRecords, 'createdAt');
    },
    dateParams() {
      return {
        // startDate: moment(this.date.start).format('YYYY-MM-DDTHH:mm:ss'),
        // endDate: moment(this.date.end).format('YYYY-MM-DDTHH:mm:ss'),
        startDate: this.date?.start,
        endDate: this.date?.end,
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
    getPages() {
      if (this.mpayCobrandCards.length === 0) return 0;
      return Math.ceil(this.mpayCobrandCards.length / this.perPage);
    },
  },
  async mounted() {
    this.isLoading = true;
    const [mpayCobrandCards] = await MpayCobrandRequests.getMpayCobrandCardStatement(
      {
        startDate: this.date.start,
        endDate: this.date.end,
      },
    );
    this.mpayCobrandCards = mpayCobrandCards.data.response;
    this.isLoading = false;
  },
  methods: {
    updatePage() {
      const startIndex = this.perPage * (this.currentPage - 1);
      const endIndex = startIndex + this.perPage;
      return this.mpayCobrandCards.slice(startIndex, endIndex);
    },
    async changePage() {
      let params;
      this.isFiltered = Object.values(this.dateParams).some((value) => value !== null);
      if (this.isFiltered) {
        this.isLoading = true;
        params = {
          startDate: moment(this.dateParams.startDate).format('YYYY-MM-DDTHH:mm:ss'),
          endDate: moment(this.dateParams.endDate).format('YYYY-MM-DDTHH:mm:ss'),
        };
      }
      this.mpayCobrandCards = await MpayCobrandRequests.getMpayCobrandCardStatement(params).then((result) => result[0].data.response);
      this.isLoading = false;
    },
    submitFilter() {
      this.changePage();
    },
    goToReceipt(item) {
      switch (item.orderType) {
        case 'TRANSFER':
          this.$router.push({
            name: 'transfer-receipt',
            params: { id: item.orderUuid },
          });
          break;
        case 'TRANSFER_TARGET':
          this.$router.push({
            name: 'transfer-receipt',
            params: { id: item.orderUuid },
          });
          break;
        case 'PAYMENT':
          this.$router.push({
            name: 'payment-receipt',
            params: { id: item.orderUuid },
          });
          break;
        case 'TOP_UP':
          this.$router.push({
            name: 'top-up-receipt',
            params: { id: item.orderUuid },
          });
          break;
        default:
          break;
      }
    },
    getAmount(item) {
      if (item.amount > 0) {
        return `+${item.amount?.toFixed(2)}`;
      }
      return `${item.amount?.toFixed(2)}`;
    },
  },
};
</script>

<style lang="scss" scoped>
@import '@/styles/components/empty.scss';
@import '@/styles/components/filter-form.scss';
@import '@/styles/components/pagination.scss';

.list-title{
  color: #262626;
  font-size: 18px;
  font-weight: 500;
  line-height: 150%; /* 27px */
  padding-bottom: 10px;
  border-bottom: 1px solid #F0F0F0;
}

.empty-page_img {
  width: 113px;
  height: 80px;
}

.filter-form {
  padding-top: 20px;
}

</style>
