<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue';
import { ChevronUpDownIcon, ArrowsRightLeftIcon, MagnifyingGlassIcon, ArrowPathIcon } from '@heroicons/vue/24/outline';
import axios from 'axios';

const amount = ref(1);
const fromCurrency = ref('USD');
const toCurrency = ref('THB');
const exchangeRate = ref(0);
const convertedAmount = ref(0);
const loadingExchangeRate = ref(false);
const loadingConversion = ref(false);
const error = ref('');
const showFromDropdown = ref(false);
const showToDropdown = ref(false);
const searchQuery = ref('');

interface Currency {
  code: string;
  symbol: string;
  flag: string;
  name: string;
}

const currencies: Currency[] = [
  { code: 'USD', symbol: '$', flag: '🇺🇸', name: 'US Dollar' },
  { code: 'EUR', symbol: '€', flag: '🇪🇺', name: 'Euro' },
  { code: 'GBP', symbol: '£', flag: '🇬🇧', name: 'British Pound' },
  { code: 'JPY', symbol: '¥', flag: '🇯🇵', name: 'Japanese Yen' },
  { code: 'THB', symbol: '฿', flag: '🇹🇭', name: 'Thai Baht' },
  { code: 'CNY', symbol: '¥', flag: '🇨🇳', name: 'Chinese Yuan' },
  { code: 'KRW', symbol: '₩', flag: '🇰🇷', name: 'South Korean Won' },
  { code: 'INR', symbol: '₹', flag: '🇮🇳', name: 'Indian Rupee' },
];

const getCurrencySymbol = (code: string) => {
  return currencies.find(c => c.code === code)?.symbol || '';
};

const selectCurrency = (type: 'from' | 'to', currency: Currency) => {
  if (type === 'from') {
    fromCurrency.value = currency.code;
    showFromDropdown.value = false; // Close dropdown after selection
  } else {
    toCurrency.value = currency.code;
    showToDropdown.value = false;
  }
};

const filteredCurrencies = computed(() => {
  if (!searchQuery.value) return currencies;
  const query = searchQuery.value.toLowerCase();
  return currencies.filter(
    currency =>
      currency.code.toLowerCase().includes(query) ||
      currency.name.toLowerCase().includes(query)
  );
});

const fetchExchangeRate = async () => {
  try {
    loadingExchangeRate.value = true;
    error.value = '';
    const response = await axios.get(`http://localhost:3000/exchange?curr1=${fromCurrency.value}&curr2=${toCurrency.value}`);
    exchangeRate.value = response.data.rate;
  } catch (err) {
    error.value = 'Failed to fetch exchange rate. Please try again later.';
  } finally {
    loadingExchangeRate.value = false;
  }
};

const calculateConversion = async () => {
  if (!amount.value || isNaN(amount.value) || amount.value <= 0) {
    error.value = 'Please enter a valid amount.';
    return;
  }

  try {
    loadingConversion.value = true;
    error.value = '';
    const response = await axios.get(
      `http://localhost:3000/exchange/amount?curr1=${fromCurrency.value}&curr2=${toCurrency.value}&amount=${amount.value}`
    );
    convertedAmount.value = response.data.rate;
  } catch (err) {
    error.value = 'Failed to calculate conversion. Please try again later.';
  } finally {
    loadingConversion.value = false;
  }
};

const swapCurrencies = () => {
  const temp = fromCurrency.value;
  fromCurrency.value = toCurrency.value;
  toCurrency.value = temp;
};

const toggleFromDropdown = () => {
  showFromDropdown.value = !showFromDropdown.value;
  showToDropdown.value = false; // Close the other dropdown
};

const toggleToDropdown = () => {
  showToDropdown.value = !showToDropdown.value;
  showFromDropdown.value = false; // Close the other dropdown
};

watch([fromCurrency, toCurrency], fetchExchangeRate);
onMounted(fetchExchangeRate);
</script>


<template>
  <div class="currency-converter">
    <h1>Currency Converter</h1>

    <div class="exchange-rate-container">
      <div class="exchange-rate">
        <span>1 {{ fromCurrency }} = {{ exchangeRate }} {{ toCurrency }}</span>
      </div>
      <button @click="fetchExchangeRate" class="update-btn" :disabled="loadingExchangeRate">
        <ArrowPathIcon class="h-4 w-4" /> Update Rate
      </button>
    </div>

    <div class="converter-form">
      <!-- Amount Input -->
      <div class="input-group">
        <label>Amount</label>
        <div class="input-with-currency">
          <span class="currency-symbol">{{ getCurrencySymbol(fromCurrency) }}</span>
          <input 
            type="number" 
            v-model="amount" 
            min="0" 
            step="0.01"
            @keyup.enter.prevent="calculateConversion"
            :disabled="loadingConversion"
          />
          <div class="currency-select" @click="toggleFromDropdown">
            <span class="currency-flag">{{ fromCurrency }}</span>
            <ChevronUpDownIcon class="h-5 w-5" />
          </div>
        </div>

        <!-- New Dropdown for selecting currency -->
        <div v-if="showFromDropdown" class="currency-dropdown">
          <input type="text" v-model="searchQuery" placeholder="Search currency..." @click.stop />
          <div class="currency-list">
            <div
              v-for="currency in filteredCurrencies"
              :key="currency.code"
              class="currency-option"
              @click="selectCurrency('from', currency)"
            >
              <span class="currency-flag">{{ currency.flag }}</span>
              <span class="currency-code">{{ currency.code }}</span>
              <span class="currency-name">{{ currency.name }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Swap Button -->
      <button @click="swapCurrencies" class="swap-btn" :disabled="loadingConversion">
        <ArrowsRightLeftIcon class="h-6 w-6 text-gray-700" />
      </button>

      <!-- Converted Amount -->
      <div class="input-group">
        <label>Converted to</label>
        <div class="input-with-currency">
          <span class="currency-symbol">{{ getCurrencySymbol(toCurrency) }}</span>
          <input 
            type="text" 
            v-model="convertedAmount" 
            disabled 
            :class="{ loading: loadingConversion }"
          />
          <div class="currency-select" @click="toggleToDropdown">
            <span class="currency-flag">{{ toCurrency }}</span>
            <ChevronUpDownIcon class="h-5 w-5" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>





<style scoped>

/* Main container */
.currency-converter {
  max-width: 700px;
  margin: 2rem auto;
  padding: 1.5rem;
  background: #f8fafc; /* Light gray background */
  border-radius: 12px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
}

/* Title */
h1 {
  font-size: 1.5rem;
  font-weight: bold;
  color: #34495e !important;  /* ✅ Darker color for contrast */
}

/* Currency Dropdown */
.currency-dropdown {
  position: absolute;
  background: white;
  border-radius: 10px;
  box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.15);
  width: 280px;
  z-index: 1000;
  top: 100%;
  left: 0;
  max-height: 300px;
  overflow-y: auto;
}

/* Search Bar */
.search-box {
  width: 100%;
  padding: 10px;
  border: none;
  font-size: 0.9rem;
  background: #f7f7f7;
  border-radius: 10px 10px 0 0;
  outline: none;
}

/* Currency List */
.currency-list {
  max-height: 250px;
  overflow-y: auto;
}

/* Currency Option */
.currency-option {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 15px;
  cursor: pointer;
  transition: background 0.2s ease-in-out;
}

.currency-option:hover {
  background: #f3f4f6;
}

/* Selected Currency */
.selected {
  background: #eaf3ff;
  font-weight: bold;
}

/* Currency Flag */
.currency-flag {
  font-size: 20px;
}

/* Currency Code */
.currency-code {
  font-weight: bold;
}

/* Checkmark */
.checkmark {
  margin-left: auto;
  color: #27ae60;
  font-weight: bold;
}


.exchange-rate-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}



.exchange-rate {
  display: flex;
  align-items: center;
  gap: 0.5rem;  /* ✅ Adds space between text and button */
  background: #ffefe6; /* Light orange */
  color: #d35400; /* Dark orange text */
  padding: 0.3rem 0.8rem; /* ✅ Smaller padding */
  border-radius: 20px; /* ✅ Makes it rounded */
  font-size: 0.9rem; /* ✅ Make text smaller */
  font-weight: 600;
  max-width: fit-content; /* ✅ Keeps it compact */
}

.update-btn {
  display: flex;
  align-items: center;
  gap: 0.3rem; /* ✅ Space between icon & text */
  background: #e67e22; /* Orange */
  color: white;
  border: none;
  padding: 0.3rem 0.8rem; /* ✅ Smaller padding */
  font-size: 0.85rem; /* ✅ Smaller text */
  font-weight: 600;
  border-radius: 4px;
  cursor: pointer;
}

.update-btn:hover {
  background: #d35400; /* Darker orange on hover */
}

/* ✅ Reduce icon size inside the button */
.update-btn svg {
  width: 14px;
  height: 14px;
}


/* Form container */
.converter-form {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-top: 1.5rem;
  max-width: 100%;
}


/* Input group */
.input-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start; /* ✅ Aligns labels to the left */
  gap: 0.5rem;
  overflow: visible;
}


/* Label */
.input-group label {
  font-size: 0.9rem;
  font-weight: bold;
  color: #34495e;
}

.input-with-currency {
  display: flex;
  align-items: center;
  position: relative; /* ✅ Ensure it is positioned relative to this container */
  background: white;
  border: 1px solid #ffffff;
  border-radius: 8px;
  width: 100%;
  max-width: 100%;
  transition: all 0.2s ease-in-out;
}


/* ✅ Readonly/Disabled Output Field */
input:disabled {
  background: #ffffff !important; /* ✅ Light gray background */
  color: #ffffff !important; /* ✅ Darker text for readability */
  font-weight: 600;
  border: 1px solid #ffffff;
}


/* Input fields */
input {
  flex: 1;
  padding: 0.75rem;
  border: none;
  font-size: 1rem;
  color: #2c3e50;
  background-color: white;
  outline: none;
  text-align: left;
  max-width: 100%; /* ✅ Ensures it doesn't exceed container */
}


.search-box {
  position: relative;
  padding: 0.5rem;
  border-bottom: 1px solid #eee;
  background-color: #fff;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

/* Target the icon specifically */
.search-box svg {
  width: 18px;     /* Smaller icon */
  height: 18px;
  color: #999;
}

/* Currency symbol inside input */
.currency-symbol, .currency-select {
  background: #ffffff;
  color: #2c3e50;
  font-weight: bold;
  padding: 0.75rem;
  border-right: 1px solid #ffffff;
}



.currency-select {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  background: #ffffff;
  cursor: pointer;
}

.currency-select:hover {
  background: #d5dbdb;
}

.swap-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f7f8fa;
  border: 1px solid #dcdfe3;
  padding: 0.75rem;
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.2s;
  position: relative;
  top: 4px;  /* ✅ Moves it slightly downward */
}

.swap-btn:hover {
  background: #e5e7eb;
}


.swap-btn svg {
  width: 20px; /* ✅ Icon size */
  height: 20px;
  color: #2c3e50;
}


/* Disabled inputs */
input:disabled {
  background: #ffffff !important; /* ✅ Light gray background */
  color: #34495e !important; /* ✅ Darker text for readability */
  font-weight: bold;
}


/* Loading animation */
.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #e67e22;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* Error message */
.error {
  color: #e74c3c;
  text-align: center;
  margin-top: 1rem;
  padding: 0.75rem;
  background: #fdecea;
  border-radius: 5px;
}


</style>