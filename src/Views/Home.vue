<template>
  <nav class="header noPrint">
    <div class="text ml-5">
      Upwork Visualization
    </div>
    <a class="source" href="https://github.com/lablnet/upwork_visualization">Source Code</a>

  </nav>
  <main class="mt-5 justify-content-center">
    <div class="container-fluid">
      <div class="" v-if="!loaded">
        <h1>Drop your file here.</h1>
        <div class="form-group">
          <label>File</label>
          <input type="file" v-on:change="readfile" class="form-control-file"/>
          <span class="text-danger" v-if="error">{{ error }}</span>
        </div>
        <h1>How to get transaction history CSV?</h1>
        <a target="_blank" href="https://support.upwork.com/hc/en-us/articles/211068188-Invoices-and-Transactions"> Follow the follwing guide</a>
      </div>
      <div class="col" v-if="loaded">
        <a style="float:left" href="javascript:void(0);" @click="clear" class="noPrint">Clear</a>
        <a style="float:right" href="javascript:window.print()" class="noPrint">print this page</a>
        <br />
        <h1>Here's is your report.</h1>
      </div>
      <div class="" v-if="loaded">
        <div>
          <h1 class="h2">Chart:</h1>
          <div style="display: flex;flex-direction:column;" >
            <EarningChart
              canvas-id="chart1"
              :labels="labels"
              :datasets="dataset"
            ></EarningChart>
          </div>
        </div>

        <div
          v-if="monthlyLabels && monthlyDatasets"
        >
          <h1 class="h2">Monthly Chart:</h1>
          <div style="display: flex;flex-direction:column;" >
            <EarningChart
              canvas-id="monthlyChart"
              :labels="monthlyLabels"
              :datasets="monthlyDatasets"
            ></EarningChart>
          </div>
        </div>

        <div class="col-md-4">
          <h1 class="h2">Stats:</h1>
          <b>Balance: </b> <i>{{ total.balance  }} USD</i> <br />
          <b>Total earning: </b> <i>{{ total.earning  }} USD</i> <br />
          <b>Total withdraw: </b> <i>{{ total.withdraw  }} USD</i> <br />
          <b>Total Withdraw Fee: </b> <i>{{ total.withdrawFee  }} USD</i> <br />
          <b>Total refund: </b> <i>{{ total.refund  }} USD</i> <br />
          <b>Total Upwork Fee: </b> <i>{{ total.fee  }} USD</i> <br />
          <b>Total Spend on Upwork: </b> <i>{{ total.membership  }} USD</i> <br />
          <b>Total Payment with Card: </b> <i>{{ total.paymentWithCard  }} USD</i> <br />
        </div>
      </div>

      <div class="table-responsive" v-if="loaded">
        <h1 class="mt-3 h2">Feeds:</h1>

        <table class="table table-primary">
          <thead>
          <tr>
            <th>ID</th>
            <th>Date</th>
            <th>Type</th>
            <th>Team</th>
            <th>Description</th>
            <th>Amount</th>
          </tr>
          </thead>
          <tbody v-for="(item, idx) in earning" :key="idx" >
          <tr v-if="item['Ref ID']">
            <td>{{ item['Ref ID'] }}</td>
            <td>{{ item['Date'].replaceAll('"', "") }} </td>
            <td>{{item['Type']}}</td>
            <td>{{ item['Team'] }}</td>
            <td>{{item['Description']}}</td>
            <td>
              {{ item['Amount'] }}
            </td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </main>
  <footer :class="getFooterPosition">
    <p class="note"><b>Note: This is an open source project built using Javascript, vuejs and chartjs, no server is involved here, your file is not uploaded to any server. For confirmation you can look to source code.</b></p>
    <p class="copyright">Made with 💕 by <a target="_blank" href="https://www.upwork.com/fl/lablnet">Muhammad Umer Farooq</a>.</p>
    <a target="_blank" href="https://github.com/lablnet/upwork_visualization">Source Code</a>
  </footer>
</template>

<script>
import csvJSON from "../csv.js"
import Vue3ChartJs from '@j-t-mcc/vue3-chartjs'
import EarningChart from '../components/EarningChart.vue'
import Swal from 'sweetalert2'

export default {
  name: "Home",
  components: {
    Vue3ChartJs,
    EarningChart,
  },
  data() {
    return {
      error: null,
      labels: [],
      dataset: [],
      loading: false,
      loaded: false,
      data: {},
      earning: [],
      withdraw: [],
      serviceFee: [],
      membership: [],
      withdrawFee: [],
      payments: [],
      other: [],
      refund: [],
      validType: ['application/vnd.ms-excel', "text/csv"],
    }
  },
  methods: {
    clear() {
      Swal.fire({
        title: 'Are you sure?',
        text: "This action can not be undone",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes, clear it!'
      }).then((result) => {
        if (result.isConfirmed) {
          this.data = []
          this.loaded = false
          this.refund = []
          this.withdrawFee = []
          this.serviceFee = []
          this.dataset = []
          this.labels = []
          this.withdraw = []
          this.membership = []
          this.earning = []
        }
      })
    },
    sum(arr) {
      let sum = 0.0
      for (let item in arr) {
        sum += Math.abs(arr[item]['Amount'])
      }
      return sum
    },
    readfile(event) {
      let reader = new FileReader();
      reader.onload =  () => {
        this.data = JSON.parse(csvJSON(reader.result)).reverse()

        let earn = []
        let fee = []
        let withdraw = []
        let refund = []
        let membership = []
        let withdrawFee = []
        let labels = []
        let dataset = {
          earning: [],
          withdraw: [],
          membership: [],
          withdrawFee: [],
          serviceFee: [],
          refund: [],
        }
        let earningsPoints = [
            'Fixed Price',
            'Hourly',
            'Bonus',
            'Miscellaneous',
            'Adjustment',
            'Milestone',
            'Expense'
        ];
        let colsList = [
          ...earningsPoints,
          'Withdrawal',
          'Service Fee',
          'Refund',
          'Membership Fee',
          'Withdrawal Fee',
          'Payment'
        ]
        for (let item in this.data) {
          if (earningsPoints.includes(this.data[item]['Type']) === true) {
            earn.push(this.data[item])
            labels.push(this.data[item]['Date'].replace('"', "").replace('"', ""))
            dataset.earning.push(Math.abs(this.data[item]['Amount']))
          }
          if (this.data[item]['Type'] === 'Withdrawal') {
            withdraw.push(this.data[item])
            dataset.withdraw.push(Math.abs(this.data[item]['Amount']))
          }
          if (this.data[item]['Type'] === 'Service Fee') {
            fee.push(this.data[item])
            dataset.serviceFee.push(Math.abs(this.data[item]['Amount']))
          }
          if (this.data[item]['Type'] === 'Refund') {
            refund.push(this.data[item])
            dataset.refund.push(Math.abs(this.data[item]['Amount']))
          }
          if (this.data[item]['Type'] === 'Membership Fee') {
            membership.push(this.data[item])
            dataset.membership.push(Math.abs(this.data[item]['Amount']))
          }
          if (this.data[item]['Type'] === 'Withdrawal Fee') {
            withdrawFee.push(this.data[item])
            dataset.withdrawFee.push(Math.abs(this.data[item]['Amount']))
          } 
          if (this.data[item]['Type'] === 'Payment') {
            this.payments.push(this.data[item])
          } 
          if (!colsList.includes(this.data[item]['Type'])){
            this.other.push(this.data[item])
            console.log ("Other", this.data[item])
          }
        }

        this.labels = labels
        this.dataset = dataset
        this.earning = earn
        this.serviceFee = fee
        this.withdraw = withdraw
        this.refund = refund
        this.membership = membership
        this.withdrawFee = withdrawFee
        this.loaded = true
      };

      this.error = null
      if (this.validType.includes(event.target.files[0].type)) {
        reader.readAsBinaryString(event.target.files[0])
      } else {
        this.error = "Please, select valid file type"
      }
      // reset the input.
      event.target.value = ''
    },
  },
  computed: {
    getFooterPosition() {
      return (this.loaded) ? "noPrint" : 'fixed'
    },
    monthlyData () {
      const earning = [];
      
      let _prevMonthStr = null
      let _totalEarning = 0

      this.earning.forEach(_item => {
        const date = new Date(_item.Date.replaceAll('"', ""))
        const dateStr = date.toLocaleString('en-US', { year: 'numeric', month: 'short' });
        

        if (_prevMonthStr === dateStr) {
          _totalEarning += parseFloat(_item.Amount)
        } else {
          console.log(_prevMonthStr, _totalEarning)

          if (_prevMonthStr !== null) {
            earning.push({
              date: _prevMonthStr,
              amount: _totalEarning,
            })
          }

          _totalEarning = 0;
          _prevMonthStr = dateStr
        }

      })

      return {
        earning
      }
    },
    monthlyLabels () {
      return this.monthlyData?.earning?.map?.(_item => _item.date)
    },
    monthlyDatasets () {
      return {
        earning: this.monthlyData?.earning?.map?.(_item => _item.amount)
      }
    },
    total(x) {
      let earning = parseFloat(this.sum(this.earning))
      let refund  = parseFloat(this.sum(this.refund))
      let withdraw = parseFloat(this.sum(this.withdraw))
      let fee = parseFloat(this.sum(this.serviceFee))
      let membership = parseFloat(this.sum(this.membership))
      let withdrawFee = parseFloat(this.sum(this.withdrawFee))
      let paymentWithCard = parseFloat(this.sum(this.payments))
      let others = parseFloat(this.sum(this.other))
      console.log ("Others", others)
      console.log ("Payment with card", paymentWithCard)
      let fees = (parseFloat(withdraw + fee + (membership - paymentWithCard) + withdrawFee + refund))    
      let balance = parseFloat(earning - (fees) ).toFixed(2)
      console.log ("Balance", balance, "fees", fees, "earning", earning, (parseFloat(withdraw) + parseFloat(fee)), "t", fee)
      return {
        balance: balance,
        earning: earning.toFixed(2),
        refund: refund.toFixed(2),
        withdraw: withdraw.toFixed(2),
        fee: fee.toFixed(2),
        membership: membership.toFixed(2),
        withdrawFee: withdrawFee.toFixed(2),
        paymentWithCard: paymentWithCard.toFixed(2),
      }
    }
  }
}
</script>
