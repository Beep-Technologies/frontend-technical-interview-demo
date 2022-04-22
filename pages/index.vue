<template>
  <div class="bg-gray-100 flex items-center justify-center w-screen h-screen p-6">
    <div class="bg-white rounded-md max-w-md p-6 shadow-sm">
      <div class="flex flex-col items-center justify-center space-y-6">
        <SearchDropdownToggle
          class="w-full"
          label="Async Search"
          description="With description and custom results display"
          is-async
          :get-options="searchCurrencies"
          option-label-path="name"
          option-value-path="code"
          :selected="selectedAsyncCurrencies"
          @select="toggleAsyncCurrencies"
        >
          <template #options="{ options, selected, onMouseDown, onMouseOver, focusedOptionIndex }">
            <div
              v-for="(option, index) in options"
              :key="index"
              :class="[
                index === focusedOptionIndex ? 'bg-teal-100 text-gray-600' :
                'odd:bg-gray-50 text-gray-600 bg-white'
              ]"
              class="flex items-center justify-between px-4 py-3 leading-5 cursor-pointer focus:outline-none"
              role="menuitem"
              @mousedown.prevent="onMouseDown(option)"
              @mouseover="onMouseOver(index)"
            >
              <div class="flex flex-col flex-1">
                <span class="text-sm text-gray-600">{{ option.emoji }} {{ option.name }}</span>
                <span class="text-sm text-gray-500">Country: {{ option.code }}</span>
              </div>

              <div class="relative pl-3 pointer-events-none">
                <div class="absolute inset-0 z-200" />
                <input :checked="selected.some(x => x.code === option.code)" :value="option" type="checkbox" class="w-4 h-4 text-teal-600 transition duration-150 ease-in-out form-checkbox">
              </div>
            </div>
          </template>
        </SearchDropdownToggle>

        <SearchDropdownToggle
          class="w-full"
          label="Sync Search"
          description="With default display and search on focus"
          common-label-path="name"
          common-value-path="code"
          :options="globalCurrencies"
          :selected="selectedSyncCurrencies"
          @select="toggleSyncCurrencies"
        />
      </div>
    </div>
  </div>
</template>

<script>
import isEqual from 'lodash.isequal'
import fuzzysort from 'fuzzysort'
import SearchDropdownToggle from '~/components/SearchDropdownToggle'
export default {
  components: {
    SearchDropdownToggle
  },
  data () {
    return {
      fuzzysort,

      selectedAsyncCurrencies: [],
      selectedSyncCurrencies: [],
      globalCurrencies: [
        {
          symbol: '€',
          name: 'Euro',
          symbol_native: '€',
          code: 'EUR',
          emoji: '🇪🇺'
        },
        {
          symbol: '$',
          name: 'US Dollar',
          symbol_native: '$',
          code: 'USD',
          emoji: '🇺🇸'
        },
        {
          symbol: 'CA$',
          name: 'Canadian Dollar',
          symbol_native: '$',
          code: 'CAD',
          emoji: '🇨🇦'
        },
        {
          symbol: 'AED',
          name: 'United Arab Emirates Dirham',
          symbol_native: 'د.إ.‏',
          code: 'AED',
          emoji: '🇦🇪'
        },
        {
          symbol: 'Af',
          name: 'Afghan Afghani',
          symbol_native: '؋',
          code: 'AFN',
          emoji: '🇦🇫'
        },
        {
          symbol: 'ALL',
          name: 'Albanian Lek',
          symbol_native: 'Lek',
          code: 'ALL',
          emoji: '🇦🇱'
        },
        {
          symbol: 'AMD',
          name: 'Armenian Dram',
          symbol_native: 'դր.',
          code: 'AMD',
          emoji: '🇦🇲'
        },
        {
          symbol: 'ƒ',
          name: 'Netherlands Antillean guilder',
          symbol_native: 'ƒ',
          code: 'ANG',
          emoji: '🇧🇶'
        },
        {
          symbol: 'Kz',
          name: 'Angolan Kwanza',
          symbol_native: 'Kz',
          code: 'AOA',
          emoji: '🇦🇴'
        },
        {
          symbol: 'AR$',
          name: 'Argentine Peso',
          symbol_native: '$',
          code: 'ARS',
          emoji: '🇦🇷'
        },
        {
          symbol: 'AU$',
          name: 'Australian Dollar',
          symbol_native: '$',
          code: 'AUD',
          emoji: '🇦🇺'
        },
        {
          symbol: 'man.',
          name: 'Azerbaijani Manat',
          symbol_native: 'ман.',
          code: 'AZN',
          emoji: '🇦🇿'
        },
        {
          symbol: 'KM',
          name: 'Bosnia-Herzegovina Convertible Mark',
          symbol_native: 'KM',
          code: 'BAM',
          emoji: '🇧🇦'
        },
        {
          symbol: 'Tk',
          name: 'Bangladeshi Taka',
          symbol_native: '৳',
          code: 'BDT',
          emoji: '🇧🇩'
        },
        {
          symbol: 'BGN',
          name: 'Bulgarian Lev',
          symbol_native: 'лв.',
          code: 'BGN',
          emoji: '🇧🇬'
        },
        {
          symbol: 'BD',
          name: 'Bahraini Dinar',
          symbol_native: 'د.ب.‏',
          code: 'BHD',
          emoji: '🇧🇭'
        },
        {
          symbol: 'FBu',
          name: 'Burundian Franc',
          symbol_native: 'FBu',
          code: 'BIF',
          emoji: '🇧🇮'
        },
        {
          symbol: 'BN$',
          name: 'Brunei Dollar',
          symbol_native: '$',
          code: 'BND',
          emoji: '🇧🇳'
        },
        {
          symbol: '$',
          name: 'Bermudian Dollar',
          symbol_native: '$',
          code: 'BMD',
          emoji: '🇧🇲'
        },
        {
          symbol: 'Bs',
          name: 'Bolivian Boliviano',
          symbol_native: 'Bs',
          code: 'BOB',
          emoji: '🇧🇴'
        },
        {
          symbol: 'R$',
          name: 'Brazilian Real',
          symbol_native: 'R$',
          code: 'BRL',
          emoji: '🇧🇷'
        },
        {
          symbol: 'Nu.',
          name: 'Bhutanese Ngultrum',
          symbol_native: 'Nu.',
          code: 'BTN',
          emoji: '🇧🇹'
        },
        {
          symbol: 'BWP',
          name: 'Botswanan Pula',
          symbol_native: 'P',
          code: 'BWP',
          emoji: '🇧🇼'
        },
        {
          symbol: 'BYR',
          name: 'Belarusian Ruble',
          symbol_native: 'BYR',
          code: 'BYR',
          emoji: '🇧🇾'
        },
        {
          symbol: 'BZ$',
          name: 'Belize Dollar',
          symbol_native: '$',
          code: 'BZD',
          emoji: '🇧🇿'
        },
        {
          symbol: 'CDF',
          name: 'Congolese Franc',
          symbol_native: 'FrCD',
          code: 'CDF',
          emoji: '🇨🇩'
        },
        {
          symbol: 'CHF',
          name: 'Swiss Franc',
          symbol_native: 'CHF',
          code: 'CHF',
          emoji: '🇨🇭'
        },
        {
          symbol: 'CL$',
          name: 'Chilean Peso',
          symbol_native: '$',
          code: 'CLP',
          emoji: '🇨🇱'
        },
        {
          symbol: 'CN¥',
          name: 'Chinese Yuan',
          symbol_native: 'CN¥',
          code: 'CNY',
          emoji: '🇨🇳'
        },
        {
          symbol: 'CO$',
          name: 'Colombian Peso',
          symbol_native: '$',
          code: 'COP',
          emoji: '🇨🇴'
        },
        {
          symbol: '₡',
          name: 'Costa Rican Colón',
          symbol_native: '₡',
          code: 'CRC',
          emoji: '🇨🇷'
        },
        {
          symbol: 'CV$',
          name: 'Cape Verdean Escudo',
          symbol_native: 'CV$',
          code: 'CVE',
          emoji: '🇨🇻'
        },
        {
          symbol: 'Kč',
          name: 'Czech Republic Koruna',
          symbol_native: 'Kč',
          code: 'CZK',
          emoji: '🇨🇿'
        },
        {
          symbol: 'Fdj',
          name: 'Djiboutian Franc',
          symbol_native: 'Fdj',
          code: 'DJF',
          emoji: '🇩🇯'
        },
        {
          symbol: 'Dkr',
          name: 'Danish Krone',
          symbol_native: 'kr',
          code: 'DKK',
          emoji: '🇩🇰'
        },
        {
          symbol: 'RD$',
          name: 'Dominican Peso',
          symbol_native: 'RD$',
          code: 'DOP',
          emoji: '🇩🇴'
        },
        {
          symbol: 'DA',
          name: 'Algerian Dinar',
          symbol_native: 'د.ج.‏',
          code: 'DZD',
          emoji: '🇩🇿'
        },
        {
          symbol: 'Ekr',
          name: 'Estonian Kroon',
          symbol_native: 'kr',
          code: 'EEK',
          emoji: '🇪🇪'
        },
        {
          symbol: 'EGP',
          name: 'Egyptian Pound',
          symbol_native: 'ج.م.‏',
          code: 'EGP',
          emoji: '🇪🇬'
        },
        {
          symbol: 'Nfk',
          name: 'Eritrean Nakfa',
          symbol_native: 'Nfk',
          code: 'ERN',
          emoji: '🇪🇷'
        },
        {
          symbol: 'Br',
          name: 'Ethiopian Birr',
          symbol_native: 'Br',
          code: 'ETB',
          emoji: '🇪🇹'
        },
        {
          symbol: '£',
          name: 'Falkland Island Pound',
          symbol_native: '£',
          code: 'FKP',
          emoji: '🇫🇰'
        },
        {
          symbol: '£',
          name: 'British Pound Sterling',
          symbol_native: '£',
          code: 'GBP',
          emoji: '🇬🇧'
        },
        {
          symbol: 'GEL',
          name: 'Georgian Lari',
          symbol_native: 'GEL',
          code: 'GEL',
          emoji: '🇬🇪'
        },
        {
          symbol: 'GH₵',
          name: 'Ghanaian Cedi',
          symbol_native: 'GH₵',
          code: 'GHS',
          emoji: '🇬🇭'
        },
        {
          symbol: '£',
          name: 'Gibraltar Pound',
          symbol_native: '£',
          code: 'GIP',
          emoji: '🇬🇮'
        },
        {
          symbol: 'FG',
          name: 'Guinean Franc',
          symbol_native: 'FG',
          code: 'GNF',
          emoji: '🇬🇳'
        },
        {
          symbol: 'GTQ',
          name: 'Guatemalan Quetzal',
          symbol_native: 'Q',
          code: 'GTQ',
          emoji: '🇬🇹'
        },
        {
          symbol: 'HK$',
          name: 'Hong Kong Dollar',
          symbol_native: '$',
          code: 'HKD',
          emoji: '🇭🇰'
        },
        {
          symbol: 'HNL',
          name: 'Honduran Lempira',
          symbol_native: 'L',
          code: 'HNL',
          emoji: '🇭🇳'
        },
        {
          symbol: 'kn',
          name: 'Croatian Kuna',
          symbol_native: 'kn',
          code: 'HRK',
          emoji: '🇭🇷'
        },
        {
          symbol: 'Ft',
          name: 'Hungarian Forint',
          symbol_native: 'Ft',
          code: 'HUF',
          emoji: '🇭🇺'
        },
        {
          symbol: 'Rp',
          name: 'Indonesian Rupiah',
          symbol_native: 'Rp',
          code: 'IDR',
          emoji: '🇮🇩'
        },
        {
          symbol: '₪',
          name: 'Israeli New Sheqel',
          symbol_native: '₪',
          code: 'ILS',
          emoji: '🇮🇱'
        },
        {
          symbol: 'Rs',
          name: 'Indian Rupee',
          symbol_native: '₹',
          code: 'INR',
          emoji: '🇮🇳'
        },
        {
          symbol: 'IQD',
          name: 'Iraqi Dinar',
          symbol_native: 'د.ع.‏',
          code: 'IQD',
          emoji: '🇮🇶'
        },
        {
          symbol: 'IRR',
          name: 'Iranian Rial',
          symbol_native: '﷼',
          code: 'IRR',
          emoji: '🇮🇷'
        },
        {
          symbol: 'Ikr',
          name: 'Icelandic Króna',
          symbol_native: 'kr',
          code: 'ISK',
          emoji: '🇮🇸'
        },
        {
          symbol: 'J$',
          name: 'Jamaican Dollar',
          symbol_native: '$',
          code: 'JMD',
          emoji: '🇯🇲'
        },
        {
          symbol: 'JD',
          name: 'Jordanian Dinar',
          symbol_native: 'د.أ.‏',
          code: 'JOD',
          emoji: '🇯🇴'
        },
        {
          symbol: '¥',
          name: 'Japanese Yen',
          symbol_native: '￥',
          code: 'JPY',
          emoji: '🇯🇵'
        },
        {
          symbol: 'с',
          name: 'Kyrgyzstani som',
          symbol_native: 'с',
          code: 'KES',
          emoji: '🇰🇪'
        },
        {
          symbol: 'KHR',
          name: 'Cambodian Riel',
          symbol_native: '៛',
          code: 'KHR',
          emoji: '🇰🇭'
        },
        {
          symbol: 'CF',
          name: 'Comorian Franc',
          symbol_native: 'FC',
          code: 'KMF',
          emoji: '🇰🇲'
        },
        {
          symbol: '₩',
          name: 'South Korean Won',
          symbol_native: '₩',
          code: 'KRW',
          emoji: '🇰🇷'
        },
        {
          symbol: 'KD',
          name: 'Kuwaiti Dinar',
          symbol_native: 'د.ك.‏',
          code: 'KWD',
          emoji: '🇰🇼'
        },
        {
          symbol: '$',
          name: 'Cayman Islands dollar',
          symbol_native: '$‏',
          code: 'KYD',
          emoji: '🇰🇾'
        },
        {
          symbol: 'KZT',
          name: 'Kazakhstani Tenge',
          symbol_native: 'тңг.',
          code: 'KZT',
          emoji: '🇰🇿'
        },
        {
          symbol: '₭',
          name: 'Lao kip',
          symbol_native: '₭‏',
          code: 'LAK',
          emoji: '🇱🇦'
        },
        {
          symbol: 'LB£',
          name: 'Lebanese Pound',
          symbol_native: 'ل.ل.‏',
          code: 'LBP',
          emoji: '🇱🇧'
        },
        {
          symbol: 'SLRs',
          name: 'Sri Lankan Rupee',
          symbol_native: 'SL Re',
          code: 'LKR',
          emoji: '🇱🇰'
        },
        {
          symbol: '$',
          name: 'Liberian Dollar',
          symbol_native: '$',
          code: 'LRD',
          emoji: '🇱🇷'
        },
        {
          symbol: 'Lt',
          name: 'Lithuanian Litas',
          symbol_native: 'Lt',
          code: 'LTL',
          emoji: '🇱🇹'
        },
        {
          symbol: 'Ls',
          name: 'Latvian Lats',
          symbol_native: 'Ls',
          code: 'LVL',
          emoji: '🇱🇻'
        },
        {
          symbol: 'LD',
          name: 'Libyan Dinar',
          symbol_native: 'د.ل.‏',
          code: 'LYD',
          emoji: '🇱🇾'
        },
        {
          symbol: 'MAD',
          name: 'Moroccan Dirham',
          symbol_native: 'د.م.‏',
          code: 'MAD',
          emoji: '🇲🇦'
        },
        {
          symbol: 'MDL',
          name: 'Moldovan Leu',
          symbol_native: 'MDL',
          code: 'MDL',
          emoji: '🇲🇩'
        },
        {
          symbol: 'MGA',
          name: 'Malagasy Ariary',
          symbol_native: 'MGA',
          code: 'MGA',
          emoji: '🇲🇬'
        },
        {
          symbol: 'MKD',
          name: 'Macedonian Denar',
          symbol_native: 'MKD',
          code: 'MKD',
          emoji: '🇲🇰'
        },
        {
          symbol: 'MMK',
          name: 'Myanma Kyat',
          symbol_native: 'K',
          code: 'MMK',
          emoji: '🇲🇲'
        },
        {
          symbol: 'MOP$',
          name: 'Macanese Pataca',
          symbol_native: 'MOP$',
          code: 'MOP',
          emoji: '🇲🇴'
        },
        {
          symbol: 'MURs',
          name: 'Mauritian Rupee',
          symbol_native: 'MURs',
          code: 'MUR',
          emoji: '🇲🇺'
        },
        {
          symbol: 'MK',
          name: 'Malawian Kwacha',
          symbol_native: 'MK',
          code: 'MWK',
          emoji: '🇲🇼'
        },
        {
          symbol: 'MX$',
          name: 'Mexican Peso',
          symbol_native: '$',
          code: 'MXN',
          emoji: '🇲🇽'
        },
        {
          symbol: 'RM',
          name: 'Malaysian Ringgit',
          symbol_native: 'RM',
          code: 'MYR',
          emoji: '🇲🇾'
        },
        {
          symbol: 'MTn',
          name: 'Mozambican Metical',
          symbol_native: 'MTn',
          code: 'MZN',
          emoji: '🇲🇿'
        },
        {
          symbol: 'N$',
          name: 'Namibian Dollar',
          symbol_native: 'N$',
          code: 'NAD',
          emoji: '🇳🇦'
        },
        {
          symbol: '₦',
          name: 'Nigerian Naira',
          symbol_native: '₦',
          code: 'NGN',
          emoji: '🇳🇬'
        },
        {
          symbol: 'C$',
          name: 'Nicaraguan Córdoba',
          symbol_native: 'C$',
          code: 'NIO',
          emoji: '🇳🇮'
        },
        {
          symbol: 'Nkr',
          name: 'Norwegian Krone',
          symbol_native: 'kr',
          code: 'NOK',
          emoji: '🇳🇴'
        },
        {
          symbol: 'NPRs',
          name: 'Nepalese Rupee',
          symbol_native: 'नेरू',
          code: 'NPR',
          emoji: '🇳🇵'
        },
        {
          symbol: 'NZ$',
          name: 'New Zealand Dollar',
          symbol_native: '$',
          code: 'NZD',
          emoji: '🇳🇿'
        },
        {
          symbol: 'OMR',
          name: 'Omani Rial',
          symbol_native: 'ر.ع.‏',
          code: 'OMR',
          emoji: '🇴🇲'
        },
        {
          symbol: 'B/.',
          name: 'Panamanian Balboa',
          symbol_native: 'B/.',
          code: 'PAB',
          emoji: '🇵🇦'
        },
        {
          symbol: 'S/.',
          name: 'Peruvian Nuevo Sol',
          symbol_native: 'S/.',
          code: 'PEN',
          emoji: '🇵🇪'
        },
        {
          symbol: '₱',
          name: 'Philippine Peso',
          symbol_native: '₱',
          code: 'PHP',
          emoji: '🇵🇭'
        },
        {
          symbol: 'PKRs',
          name: 'Pakistani Rupee',
          symbol_native: '₨',
          code: 'PKR',
          emoji: '🇵🇰'
        },
        {
          symbol: 'zł',
          name: 'Polish Zloty',
          symbol_native: 'zł',
          code: 'PLN',
          emoji: '🇵🇱'
        },
        {
          symbol: '₲',
          name: 'Paraguayan Guarani',
          symbol_native: '₲',
          code: 'PYG',
          emoji: '🇵🇾'
        },
        {
          symbol: 'QR',
          name: 'Qatari Rial',
          symbol_native: 'ر.ق.‏',
          code: 'QAR',
          emoji: '🇶🇦'
        },
        {
          symbol: 'RON',
          name: 'Romanian Leu',
          symbol_native: 'RON',
          code: 'RON',
          emoji: '🇷🇴'
        },
        {
          symbol: 'din.',
          name: 'Serbian Dinar',
          symbol_native: 'дин.',
          code: 'RSD',
          emoji: '🇷🇸'
        },
        {
          symbol: 'RUB',
          name: 'Russian Ruble',
          symbol_native: 'руб.',
          code: 'RUB',
          emoji: '🇷🇺'
        },
        {
          symbol: 'RWF',
          name: 'Rwandan Franc',
          symbol_native: 'FR',
          code: 'RWF',
          emoji: '🇷🇼'
        },
        {
          symbol: 'SR',
          name: 'Saudi Riyal',
          symbol_native: 'ر.س.‏',
          code: 'SAR',
          emoji: '🇸🇦'
        },
        {
          symbol: '$',
          name: 'Solomon Islander Dollar',
          symbol_native: '$',
          code: 'SBD',
          emoji: '🇸🇧'
        },
        {
          symbol: 'SDG',
          name: 'Sudanese Pound',
          symbol_native: 'SDG',
          code: 'SDG',
          emoji: '🇸🇩'
        },
        {
          symbol: 'Skr',
          name: 'Swedish Krona',
          symbol_native: 'kr',
          code: 'SEK',
          emoji: '🇸🇪'
        },
        {
          symbol: 'S$',
          name: 'Singapore Dollar',
          symbol_native: '$',
          code: 'SGD',
          emoji: '🇸🇬'
        },
        {
          symbol: 'Le',
          name: 'Sierra Leonean Leone',
          symbol_native: 'Le',
          code: 'SLL',
          emoji: '🇸🇱'
        },
        {
          symbol: 'Ssh',
          name: 'Somali Shilling',
          symbol_native: 'Ssh',
          code: 'SOS',
          emoji: '🇸🇴'
        },
        {
          symbol: '£',
          name: 'South Sudanese pound',
          symbol_native: '£',
          code: 'SSP',
          emoji: '🇸🇸'
        },
        {
          symbol: 'Db',
          name: 'Sao Tomean Dobra',
          symbol_native: 'Db',
          code: 'STN',
          emoji: '🇸🇹'
        },
        {
          symbol: 'SY£',
          name: 'Syrian Pound',
          symbol_native: 'ل.س.‏',
          code: 'SYP',
          emoji: '🇸🇾'
        },
        {
          symbol: 'L',
          name: 'Swazi Lilangeni',
          symbol_native: 'L‏',
          code: 'SZL',
          emoji: '🇸🇿'
        },
        {
          symbol: '฿',
          name: 'Thai Baht',
          symbol_native: '฿',
          code: 'THB',
          emoji: '🇹🇭'
        },
        {
          symbol: 'ЅМ',
          name: 'Tajikistani Somoni',
          symbol_native: 'ЅМ',
          code: 'TJS',
          emoji: '🇹🇭'
        },
        {
          symbol: 'DT',
          name: 'Tunisian Dinar',
          symbol_native: 'د.ت.‏',
          code: 'TND',
          emoji: '🇹🇳'
        },
        {
          symbol: 'T$',
          name: 'Tongan Paʻanga',
          symbol_native: 'T$',
          code: 'TOP',
          emoji: '🇹🇴'
        },
        {
          symbol: 'TL',
          name: 'Turkish Lira',
          symbol_native: 'TL',
          code: 'TRY',
          emoji: '🇹🇷'
        },
        {
          symbol: 'TT$',
          name: 'Trinidad and Tobago Dollar',
          symbol_native: '$',
          code: 'TTD',
          emoji: '🇹🇹'
        },
        {
          symbol: 'NT$',
          name: 'New Taiwan Dollar',
          symbol_native: 'NT$',
          code: 'TWD',
          emoji: '🇹🇼'
        },
        {
          symbol: 'TSh',
          name: 'Tanzanian Shilling',
          symbol_native: 'TSh',
          code: 'TZS',
          emoji: '🇹🇿'
        },
        {
          symbol: '₴',
          name: 'Ukrainian Hryvnia',
          symbol_native: '₴',
          code: 'UAH',
          emoji: '🇺🇦'
        },
        {
          symbol: 'USh',
          name: 'Ugandan Shilling',
          symbol_native: 'USh',
          code: 'UGX',
          emoji: '🇺🇬'
        },
        {
          symbol: '$U',
          name: 'Uruguayan Peso',
          symbol_native: '$',
          code: 'UYU',
          emoji: '🇺🇾'
        },
        {
          symbol: 'UZS',
          name: 'Uzbekistan Som',
          symbol_native: 'UZS',
          code: 'UZS',
          emoji: '🇺🇿'
        },
        {
          symbol: 'Bs.F.',
          name: 'Venezuelan Bolívar',
          symbol_native: 'Bs.F.',
          code: 'VEF',
          emoji: '🇻🇪'
        },
        {
          symbol: '₫',
          name: 'Vietnamese Dong',
          symbol_native: '₫',
          code: 'VND',
          emoji: '🇻🇳'
        },
        {
          symbol: 'Vt',
          name: 'Ni-Vanuatu Vatu',
          symbol_native: 'Vt',
          code: 'VUV',
          emoji: '🇻🇺'
        },
        {
          symbol: 'FCFA',
          name: 'CFA Franc BEAC',
          symbol_native: 'FCFA',
          code: 'XAF',
          emoji: '🇨🇫'
        },
        {
          symbol: '$',
          name: 'East Caribbean Dollar',
          symbol_native: '$',
          code: 'XCD',
          emoji: '🇦🇬'
        },
        {
          symbol: 'CFA',
          name: 'CFA Franc BCEAO',
          symbol_native: 'CFA',
          code: 'XOF',
          emoji: '🇨🇮'
        },
        {
          symbol: 'Fr',
          name: 'CFP franc',
          symbol_native: 'Fr',
          code: 'XPF',
          emoji: '🇵🇫'
        },
        {
          symbol: 'YR',
          name: 'Yemeni Rial',
          symbol_native: 'ر.ي.‏',
          code: 'YER',
          emoji: '🇾🇪'
        },
        {
          symbol: 'R',
          name: 'South African Rand',
          symbol_native: 'R',
          code: 'ZAR',
          emoji: '🇿🇦'
        },
        {
          symbol: 'ZK',
          name: 'Zambian Kwacha',
          symbol_native: 'ZK',
          code: 'ZMK',
          emoji: '🇿🇲'
        }
      ]
    }
  },
  mounted () {
    console.log(this.convertRemToPixels(1))
  },
  methods: {
    isEqual,

    sleep (ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },

    async searchCurrencies (query) {
      await this.sleep(500)
      // return array.slice(0, array.length > 10 ? 10 : array.length)
      const array = fuzzysort.go(query, this.globalCurrencies, { key: 'name' })
      return array.slice(0, array.length > 10 ? 10 : array.length).sort((a, b) => b.score - a.score).map(item => item.obj)
    },

    toggleAsyncCurrencies (option) {
      if (this.selectedAsyncCurrencies.some(currency => currency.name === option.name)) {
        this.selectedAsyncCurrencies = this.selectedAsyncCurrencies.filter(currency => currency.name !== option.name)
      } else {
        this.selectedAsyncCurrencies = [...this.selectedAsyncCurrencies, option]
      }
    },

    toggleSyncCurrencies (option) {
      if (this.selectedSyncCurrencies.some(currency => currency.name === option.name)) {
        this.selectedSyncCurrencies = this.selectedSyncCurrencies.filter(currency => currency.name !== option.name)
      } else {
        this.selectedSyncCurrencies = [...this.selectedSyncCurrencies, option]
      }
    },

    convertRemToPixels (rem) {
      return rem * parseFloat(getComputedStyle(document.documentElement).fontSize)
    }

  }
}
</script>

<style>
</style>
