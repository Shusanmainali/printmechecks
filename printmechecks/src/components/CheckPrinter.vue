<template>
  <div class="wrapper" id="wrapper" style="position: relative">
    <div class="check-box" id="check-box">
      <div style="position: relative" id="check-box-print">
        <div class="account-holder-name" style="position: absolute; top: 40px; left: 300px">
          {{ check.accountHolderName }}
        </div>

        <div class="date-data" style="position: absolute; top: 80px; left: 850px">
          {{ check.date }}
        </div>
        <div class="amount-box" style="position: absolute; top: 175px; left: 950px"></div>
        <div class="amount-data" style="position: absolute; top: 182px; left: 970px">
          {{ formatMoney(check.amount) }}
        </div>
        <div class="pay-to-data" style="position: absolute; top: 180px; left: 180px">
          {{ check.payTo }}
        </div>
        <div class="pay-to" style="position: absolute; top: 170px; left: 60px">
          Pay Against this  <br />Cheque to <span class="payto-line"></span>
        </div>
        <div
          class="amount-line-data"
          ref="line"
          style="position: absolute; top: 240px; left: 100px"
        >
          ***
          {{ toWords(check.amount) }}
          ***
        </div>
        <div class="amount-line" style="position: absolute; top: 250px; left: 60px">
          <span class="dollar-line"></span>
        </div>
      </div>
    </div>
    <div class="check-data" style="position: absolute; top: 450px">
      <button type="button" style="float: right" class="btn btn-primary" @click="printCheck">
        Print (Ctrl + P)
      </button>

      <!-- New button to print another check -->
      <button
        type="button"
        style="float: right; margin-right: 10px"
        class="btn btn-secondary"
        @click="resetCheck"
      >
        Print Another Check
      </button>

      <form class="row g-3" style="margin-top: 30px; border-top: 1px solid #e7e7e7"></form>
      <form class="row g-3" style="margin-top: 30px; border-top: 1px solid #e7e7e7">
        <div class="col-md-2">
          <label for="inputEmail4" class="form-label">Amount</label>
          <input type="email" class="form-control" id="inputEmail4" v-model="check.amount" />
        </div>
        <div class="col-md-6">
          <label for="inputZip" class="form-label">Pay To</label>
          <input type="text" class="form-control" v-model="check.payTo" />
        </div>
        <div class="col-md-2">
          <label for="inputEmail4" class="form-label">Date</label>
          <input type="email" class="form-control" id="inputEmail4" v-model="check.date" />
        </div>

        <div class="col-md-2">
          <label for="inputAccountHolder" class="form-label">To Pay or not</label>
          <input
            type="text"
            class="form-control"
            id="inputAccountHolder"
            v-model="check.accountHolderName"
          />
        </div>
      </form>
      <div class="col-12" style="margin-top: 30px">
        <button type="button" class="btn btn-primary" @click="saveToHistory">
          Save to History
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import print from 'print-js'
import { ToWords } from 'to-words'
import { ref, reactive, nextTick, watch, onMounted, onUnmounted } from 'vue'
import { formatMoney } from '../utilities.ts'
import { useAppStore } from '../stores/app.ts'

const state = useAppStore()

const toWordsTool = new ToWords({
  localeCode: 'en-US',
  converterOptions: {
    currency: true,
    ignoreDecimal: false,
    ignoreZeroCurrency: false,
    doNotAddOnly: true
  }
})

const toWords = (denom) => {
  try {
    let words = toWordsTool.convert(Number(denom))

    words = words.replace('Dollars', 'Rupees').replace('Cents', 'Paisa').replace('And', 'and')

    return words
  } catch (e) {
    return `${e}`
  }
}

function printCheck() {
  const style = document.createElement('style')
  style.textContent = `
      @media print {
        @page {
          margin: 0;
        }
        body {
          transform: scale(1);
          transform-origin: top center;
          width: 149%;
          margin: 0;
          padding: 0;
        }
        .wrapper > *:not(.check-box) {
          display: none !important;
        }
        .check-data {
            display: none;
        }
        .check-box {
          position: fixed;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          margin: 0;
          padding: 0px;
          background-color: white;
          background: white !important;
          border: none !important;
          box-shadow: none !important;
        }
        .check-box-print {
          position: relative;
        }
      }
    `
  document.head.appendChild(style)
  window.print()
  style.remove()
}

function saveToHistory() {
  let checkList = JSON.parse(localStorage.getItem('checkList') || '[]')
  checkList.push(check)
  localStorage.setItem('checkList', JSON.stringify(checkList))
}

function genNewCheck() {
  let checkList = JSON.parse(localStorage.getItem('checkList') || '[]')
  let recentCheck = checkList[checkList.length - 1]
  let check = {}
  check.accountHolderName = recentCheck?.accountHolderName || ''
  check.date = new Date().toLocaleDateString()
  check.amount = '0.00'
  check.payTo = ''
  return check
}

const check = reactive(genNewCheck())

const line = ref(null)

watch(
  check,
  async () => {
    await nextTick(() => {
      let computedLine = line?.value?.clientWidth
      check.lineLength = computedLine
    })
  },
  { immediate: true }
)

function handlePrintShortcut(event: KeyboardEvent) {
  if (event.ctrlKey && event.key === 'p') {
    event.preventDefault()
    printCheck()
  }
}
function resetCheck() {
  // Reset the check data to create a new check form
  Object.assign(check, genNewCheck())
}

onMounted(() => {
  if (state.check) {
    check.accountHolderName = state.check.accountHolderName
    check.accountHolderAddress = state.check.accountHolderAddress
    check.accountHolderCity = state.check.accountHolderCity
    check.accountHolderState = state.check.accountHolderState
    check.accountHolderZip = state.check.accountHolderZip
    check.checkNumber = state.check.checkNumber
    check.date = state.check.date
    check.bankName = state.check.bankName
    check.amount = state.check.amount
    check.payTo = state.check.payTo
    check.memo = state.check.memo
    check.signature = state.check.signature
    check.routingNumber = state.check.routingNumber
    check.bankAccountNumber = state.check.bankAccountNumber
  }
  state.check = null

  window.addEventListener('keydown', handlePrintShortcut)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handlePrintShortcut)
})
</script>

<style>
label {
  font-weight: bold;
}
.memo-data {
  font-family: Caveat;
  font-size: 30px;
  max-width: 350px;
  line-height: 0.65;
}
.signature-data {
  font-family: Caveat;
  font-size: 40px;
  transform: rotate(-2deg);
}
.amount-line-data {
  text-transform: capitalize;
}
.date-data,
.pay-to-data,
.amount-data {
  font-size: 20px;
  font-weight: bold;
}
.check-data {
  margin-top: 50px;
  padding: 50px 120px;
  border-top: 1px solid #e6e6e6;
}
.bank-name {
  font-size: 20px;
  font-weight: bold;
}
.account-holder-name {
  font-size: 20px;
  font-weight: bold;
}
.check-number-human {
  font-size: 20px;
  font-weight: bold;
}
.amount-box::before {
  content: '';
  font-size: 20px;
  margin-left: -15px;
  font-weight: bold;
}
.amount-box {
  width: 225px;
  height: 40px;
  border: 1px solid #c7c7c7;
  background-color: white;
}
.check-box {
  width: 1200px;
  height: 1553px;
  border: 1px solid #e6e6e6;
  background-color: white;
  margin: 0 auto;
  background: url('../assets/checkbg.png');
  background-repeat: no-repeat;
  background-size: contain;
}

#check-box {
  width: 100%;
}

@font-face {
  font-family: 'banking';
  src: url('../assets/micrenc.ttf');
}

.banking {
  font-family: 'banking';
  font-size: 37px;
}

.dollar-line {
  width: 840px;
  display: block;
  border-bottom: 1px solid black;
  margin-left: 10px;
  margin-top: 20px;
}
.payto-line {
  width: 776px;
  display: block;
  border-bottom: 1px solid black;
  margin-left: 73px;
  border-right: 1px solid black;
  height: 28px;
  margin-top: -32px;
}
</style>
