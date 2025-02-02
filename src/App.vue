<script setup lang="ts">
  import {
    NGradientText,
    NFlex,
    NCard,
    NText,
    NImage,
    NRate,
    NInput,
    NForm,
    NFormItem,
    NSpin,
    NIcon
  } from 'naive-ui'
  import { onMounted, ref } from 'vue'
  import axios from 'axios'
  import * as CryptoJS from 'crypto-js'
  import LoadingIcon from '@/components/LoadingIcon.vue'

  export interface coin {
    id: string
    symbol: string
    name: string
    image: string
    current_price: number
    market_cap: number
    market_cap_rank: number
    fully_diluted_valuation: number
    total_volume: number
    high_24h: number
    low_24h: number
    price_change_24h: number
    price_change_percentage_24h: number
    market_cap_change_24h: number
    market_cap_change_percentage_24h: number
    circulating_supply: number
    total_supply: number
    max_supply: number
    ath: number
    ath_change_percentage: number
    ath_date: string
    atl: number
    atl_change_percentage: number
    atl_date: string
    roi: string
    last_updated: string
  }

  const pickCoin = async () => {
    showCoin.value = false;
    showLoading.value = true;

    const createRequestConfig = (page: number) => ({
      method: 'GET',
      url: 'https://api.coingecko.com/api/v3/coins/markets',
      headers: {
        accept: 'application/json',

      },
      params: {
        vs_currency: 'usd',
        order: 'market_cap_desc',
        per_page: 250,  // 保持每页最大值
        page: page,     // 动态页码
        sparkline: false,
      }
    });

    try {
      const [page1Res, page2Res] = await Promise.all([
        axios.request(createRequestConfig(1)),
        axios.request(createRequestConfig(2))
      ]);

      const allCoins = [
        ...page1Res.data,
        ...page2Res.data
      ].slice(0, 500);

      const hash = CryptoJS
        .SHA256(form.value.name)
        .toString(CryptoJS.enc.Hex);

      const lastChar = hash.slice(-1);
      const lastDigit = parseInt(lastChar, 16);
      const n = lastDigit % 10;

      const startIndex = 50 * n;
      const endIndex = startIndex + 50;
      const range = allCoins.slice(startIndex, endIndex);

      myCoin.value = null
      if (range.length > 0) {
        myCoin.value = range[Math.floor(Math.random() * range.length)];
        showCoin.value = true;
      }
      showLoading.value = false;
    } catch (err) {
      console.error('Failed to fetch coins:', err);
      // 可以添加错误处理逻辑
    }
  }

  const rules = {
    name: {
      required: true,
      message: 'Please Enter Your Name',
      trigger: 'blur',
    },
  }

  const myCoin = ref<coin | null>(null)
  const showCoin = ref(false)
  const showLoading = ref(false)
  const form = ref({
    name: '',
  })

  const screenWidth = ref(document.documentElement.clientWidth)
  const isPhone = ref(screenWidth.value < 993)

  onMounted(() => {
    window.addEventListener('resize', () => {
      screenWidth.value = document.body.offsetWidth
      isPhone.value = document.body.offsetWidth < 993
    })
  })
</script>

<template>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <div class="title">
    <n-gradient-text
      gradient="linear-gradient(45deg, rgb(255, 179, 191), rgb(180, 179, 255))"
      style="font-size: 36px; font-weight: bold; margin-top: 36px"
    >
      Web3 Fortune
    </n-gradient-text>
    <br />
    <n-gradient-text
      gradient="linear-gradient(45deg, rgb(255, 179, 191), rgb(180, 179, 255))"
      style="font-size: 18px"
    >
      Discover your personalized Web3 fortune
    </n-gradient-text>
  </div>
  <n-flex class="content" justify="center">
    <n-card style="width: 25%" bordered class="card" hoverable v-if="!isPhone">
      <div style="text-align: center; margin-bottom: 24px">
        <n-image width="15" src="/static/flame.svg" preview-disabled />
        <n-text style="font-weight: bold; font-size: 18px"> Today's Hot Coins </n-text>
      </div>
      <n-flex vertical align="center" style="margin-bottom: 12px">
        <n-text style="font-weight: bold; font-size: 16px"> 1. SUI </n-text>
        <n-rate readonly :default-value="5" color="orange" />
      </n-flex>
      <n-flex vertical align="center" style="margin-bottom: 12px">
        <n-text style="font-weight: bold; font-size: 16px"> 2. BTC </n-text>
        <n-rate readonly :default-value="5" color="orange" />
      </n-flex>
      <n-flex vertical align="center" style="margin-bottom: 12px">
        <n-text style="font-weight: bold; font-size: 16px"> 3. ETF </n-text>
        <n-rate readonly :default-value="4" color="orange" />
      </n-flex>
      <n-flex vertical align="center" style="margin-bottom: 12px">
        <n-text style="font-weight: bold; font-size: 16px"> 4. DOGE </n-text>
        <n-rate readonly :default-value="3" color="orange" />
      </n-flex>
      <n-flex vertical align="center" style="margin-bottom: 12px">
        <n-text style="font-weight: bold; font-size: 16px"> 5. APT </n-text>
        <n-rate readonly :default-value="2" color="orange" />
      </n-flex>
    </n-card>
    <n-card
      :style="{
        'width': isPhone? '90%': '40%',
        'margin-left': '12px',
        'margin-right': '12px'
      }"
      bordered
      class="card"
      hoverable
    >
      <n-spin :show="showLoading" :rotate="false" :size="60">
        <template #icon>
          <n-icon>
            <loading-icon/>
          </n-icon>
        </template>
        <div style="text-align: center; margin-bottom: 24px">
          <n-image width="20" src="/static/lucky.png" preview-disabled />
          <n-text style="font-weight: bold; font-size: 18px"> Your Fortune </n-text>
        </div>
        <n-text style="font-weight: bold; font-size: 18px"> Enter Your Info </n-text>
        <n-image width="18" src="/static/finger.png" preview-disabled />
        <n-form
          label-placement="left"
          require-mark-placement="right-hanging"
          :rules="rules"
          label-width="auto"
          style="margin-top: 24px; width: 90%"
          :model="form"
        >
          <n-form-item label="Name" path="name">
            <n-input
              style="border-radius: 10px"
              placeholder="Your Name"
              v-model:value="form.name"
            />
          </n-form-item>
          <n-form-item label="Zodiac">
            <n-input style="border-radius: 10px" placeholder="Your Zodiac" />
          </n-form-item>
        </n-form>
        <n-flex vertical align="center" v-if="showCoin && myCoin != null" style="margin-top: 24px">
          <n-image width="120" :src="myCoin.image" preview-disabled />
          <n-text style="font-weight: bold; font-size: 24px">
            {{ myCoin.symbol }}
          </n-text>
          <n-text style="font-weight: bold; font-size: 16px; color: orange">
            $ {{ myCoin.current_price }}
          </n-text>
        </n-flex>
        <n-flex justify="center" style="margin-top: 24px">
          <button class="comic-button" @click="pickCoin">Get Fortune!</button>
        </n-flex>
      </n-spin>
    </n-card>
    <n-card style="width: 25%" bordered class="card" hoverable v-if="!isPhone">
      <div style="text-align: center; margin-bottom: 24px">
        <n-image width="20" src="/static/recommend.png" preview-disabled />
        <n-text style="font-weight: bold; font-size: 18px"> Recommended for Aquarius </n-text>
      </div>
    </n-card>
  </n-flex>
</template>

<style scoped>
.title {
  text-align: center;
  background-image: url('/static/title-background.png');
  height: 135vh;
  background-size: cover;
}
.content {
  position: absolute;
  top: 26vh;
  margin-left: 10%;
  width: 80%;
  border-radius: 45px;
  background-image: linear-gradient(135deg, rgb(153, 255, 206), rgb(187, 173, 255));
  height: 100vh;
  .card {
    margin-top: 24px;
    margin-bottom: 24px;
    border-radius: 25px;
    background: rgba(255, 255, 255, .7);
    -webkit-backdrop-filter: blur(5px);
    backdrop-filter: blur(5px);
  }
}
.comic-button {
  display: inline-block;
  padding: 10px 20px;
  font-size: 24px;
  font-weight: bold;
  text-align: center;
  text-decoration: none;
  color: #fff;
  background-color: rgb(187, 173, 255);
  border: 2px solid rgb(153, 255, 206);
  border-radius: 10px;
  box-shadow: 5px 5px 0 rgb(153, 255, 206);
  transition: all 0.3s ease;
  cursor: pointer;
}
.comic-button:hover {
  background-color: #fff;
  color: rgb(187, 173, 255);
  border: 2px solid rgb(187, 173, 255);
  box-shadow: 5px 5px 0 rgb(187, 173, 255);
}
.comic-button:active {
  background-color: rgb(153, 255, 206);
  box-shadow: none;
  transform: translateY(4px);
}
</style>
