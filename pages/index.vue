<template>
  <a-layout id="components-layout-demo-fixed">
    <a-layout-header :style="{ position: 'fixed', zIndex: 1, width: '100%' }">
      <img class="logo" src="~/assets/logo.png" />
      <a-menu theme="dark" mode="horizontal" :style="{ lineHeight: '64px' }">
        <a-menu-item @click="account_logout" style="float: right">
          登出
        </a-menu-item>
      </a-menu>
    </a-layout-header>
    <a-layout-content :style="{ marginTop: '64px' }">
      <a-breadcrumb :style="{ margin: '16px 20px' }">
        <a-breadcrumb-item>Profit Pocket</a-breadcrumb-item>
        <a-breadcrumb-item>{{
          viewStatus === 'login' ? account : 'Login'
        }}</a-breadcrumb-item>
      </a-breadcrumb>
      <div :style="{ background: '#fff', padding: '3%', minHeight: '380px' }">
        <a-result
          title="Login to view your profit!"
          v-show="viewStatus == 'logout'"
        >
          <template #icon>
            <a-icon type="smile" theme="twoTone" />
          </template>
          <template #extra>
            <a-form layout="inline" :form="form" @submit="handleSubmit">
              <a-form-item
                :validate-status="userNameError() ? 'error' : ''"
                :help="userNameError() || ''"
              >
                <a-input
                  v-decorator="[
                    'userName',
                    {
                      rules: [
                        {
                          required: true,
                          message: 'Please input your username!',
                        },
                      ],
                    },
                  ]"
                  placeholder="Username"
                >
                  <a-icon
                    slot="prefix"
                    type="user"
                    style="color: rgba(0, 0, 0, 0.25)"
                  />
                </a-input>
              </a-form-item>
              <a-form-item
                :validate-status="passwordError() ? 'error' : ''"
                :help="passwordError() || ''"
              >
                <a-input
                  v-decorator="[
                    'password',
                    {
                      rules: [
                        {
                          required: true,
                          message: 'Please input your Password!',
                        },
                      ],
                    },
                  ]"
                  type="password"
                  placeholder="Password"
                >
                  <a-icon
                    slot="prefix"
                    type="lock"
                    style="color: rgba(0, 0, 0, 0.25)"
                  />
                </a-input>
              </a-form-item>
              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  :disabled="hasErrors(form.getFieldsError())"
                >
                  Log in
                </a-button>
              </a-form-item>
            </a-form>
          </template>
        </a-result>
        <a-skeleton active v-show="viewStatus == 'loading'" />

        <div v-show="viewStatus == 'login'">
          <a-alert message="大家好，由於比特幣加密行情看好，九月將策略調整成「潛在獲利較高」的投資組合，最大風險預期控制在（-20%），但年化報酬預期可以超過 +50%。季底（2021-12-31）有獲利就會分潤。" type="info" closable show-icon />
          <br/>
          <a-card>
            <a-row>
              <a-col :span="24">
                <a-statistic
                  title="總資產淨值"
                  :value="info.cumulated_income + info.capital"
                  :precision="2"
                  :suffix="`USDT ( ≈ NT $${Math.round(
                    (info.cumulated_income + info.capital) * 28
                  )})`"
                  :value-style="{ color: '#3f8600' }"
                >
                </a-statistic>
              </a-col>
              <a-col :span="24" style="margin-top: 32px">
                <a-statistic
                  title="近一天盈虧"
                  :value="info.daily_income"
                  :precision="2"
                  :suffix="`USDT (${
                    Math.round((info.daily_income / info.capital) * 100000) /
                    1000
                  } %)`"
                  :value-style="{
                    color: info.daily_income > 0 ? '#ff0062' : '#3f8600',
                  }"
                >
                  <template #prefix>
                    <a-icon
                      :type="info.daily_income > 0 ? 'arrow-up' : 'arrow-down'"
                    />
                  </template>
                </a-statistic>
              </a-col>
              <a-col :span="24" style="margin-top: 32px">
                <a-statistic
                  title="累計盈虧"
                  :value="info.cumulated_income"
                  :precision="2"
                  :suffix="`USDT (${
                    Math.round(
                      (info.cumulated_income / info.capital) * 100000
                    ) / 1000
                  } %)`"
                  :value-style="{
                    color: info.cumulated_income > 0 ? '#ff0062' : '#3f8600',
                  }"
                >
                  <template #prefix>
                    <a-icon
                      :type="
                        info.cumulated_income > 0 ? 'arrow-up' : 'arrow-down'
                      "
                    />
                  </template>
                </a-statistic>
              </a-col>
              <a-col :span="24" style="margin-top: 32px">
                <a-statistic
                  title="總投入資金"
                  :value="info.capital"
                  suffix="USDT"
                />
              </a-col>
            </a-row>
            <br />
            <p>
              USDT與新台幣換算請參考<a
                href="https://max.maicoin.com/markets/usdttwd"
                target="_blank"
                >MAX平台</a
              >。
            </p>
          </a-card>
          <br />
          <h3>累計盈虧</h3>
          <Plotly
            :data="data"
            :layout="layout"
            :config="{responsive: true}"
            :display-mode-bar="false"
          ></Plotly>
        </div>
      </div>
    </a-layout-content>
    <a-layout-footer :style="{ textAlign: 'center' }">
      <img src="~/assets/logo_background.png" width="240" />
    </a-layout-footer>
  </a-layout>
</template>

<script>
import { Plotly } from 'vue-plotly'

const columns = [
  {
    title: 'Time',
    dataIndex: 'time',
  },
  {
    title: 'Info',
    dataIndex: 'info',
  },
  {
    title: 'Income',
    className: 'column-money',
    dataIndex: 'income',
  },
]

function hasErrors(fieldsError) {
  return Object.keys(fieldsError).some((field) => fieldsError[field])
}
export default {
  components: {
    Plotly,
  },
  data() {
    return {
      account: '',
      hasErrors,
      columns,
      form: this.$form.createForm(this, { name: 'horizontal_login' }),
      viewStatus: 'logout',
      info: {
        cumulated_income: 0,
        cumulated_income_percentage: 0,
        daily_income: 0,
        estimate_yearly_return: 0,
        estimate_yearly_return_percentage: 0,
        capital: 0,
        txns: [],
      },
      data: [
        {
          x: [],
          y: [],
          type: 'scatter',
          line: {shape: 'spline'}, 
          mode: 'lines'
        },
      ],
      layout: {
        yaxis: {
          tickformat: '.1%',
          showgrid: false,
          zeroline: true,
          zerolinecolor: '#eee',
          tickfont: {
            color: '#aaa'
          }
        },
        xaxis: {
          showgrid: false,
          tickfont: {
            color: '#aaa'
          }
        },
        
        height: 200,
        margin: {
          l: 50,
          r: 0,
          b: 20,
          t: 20,
          pad: 4,
        },
      },
    }
  },
  computed: {
    deadline() {
      var now = new Date()
      var arr = []
      var d1 = new Date()
      d1.setHours(8, 0, 0, 0)
      if (d1 > now) {
        arr.push(d1)
      }

      var d2 = new Date()
      d2.setHours(16, 0, 0, 0)
      if (d2 > now) {
        arr.push(d2)
      }

      var d3 = new Date()
      d3.setHours(0, 0, 0, 0)
      d3.setDate(now.getDate() + 1)
      if (d3 > now) {
        arr.push(d3)
      }

      console.log('arr', arr)

      return Math.min(...arr)
    },
  },
  mounted() {
    this.$nextTick(() => {
      // To disabled submit button at the beginning.
      this.form.validateFields()
      if (process.browser) {
        let account = localStorage.getItem('account')
        let password = localStorage.getItem('password')
        if (account !== null) {
          this.account_login(account, password)
        }
        document.querySelector('body').style.backgroundColor = '#f0f2f5'
      }
    })
  },
  methods: {
    // Only show error after a field is touched.
    userNameError() {
      const { getFieldError, isFieldTouched } = this.form
      return isFieldTouched('userName') && getFieldError('userName')
    },
    // Only show error after a field is touched.
    passwordError() {
      const { getFieldError, isFieldTouched } = this.form
      return isFieldTouched('password') && getFieldError('password')
    },
    handleSubmit(e) {
      e.preventDefault()
      let vm = this
      this.form.validateFields((err, values) => {
        if (!err) {
          vm.account_login(values['userName'], values['password'])
        }
      })
    },
    async account_login(account, password) {
      this.account = account
      if (process.browser) {
        localStorage.setItem('account', account)
        localStorage.setItem('password', password)
      }

      const hide = this.$message.loading('正在登入..', 0)
      this.viewStatus = 'loading'
      try {
        const data = await this.$axios.$post(
          'https://asia-east2-finlab-298104.cloudfunctions.net/function-2',
          { account: account, password: password }
        )
        console.log(data)
        this.$message.success('登入成功')
        this.viewStatus = 'login'
        this.info = data

        const initialCap = this.info.series['2021-09-01 11:00:08.598543']
        for (let key in this.info.series) {
          this.data[0].x.push(key)
          this.data[0].y.push((this.info.series[key] / initialCap - 1))
        }
      } catch (error) {
        this.$message.error('登入失敗')
        this.viewStatus = 'logout'
      }
      hide()
    },
    account_logout() {
      if (process.browser) {
        localStorage.removeItem('account')
        localStorage.removeItem('password')
        this.viewStatus = 'logout'
      }
    },
  },
}
</script>
<style>
#components-layout-demo-fixed .logo {
  margin-top: 15px;
  height: 32px;
  float: left;
}

h3 {
  color: #777;
}
</style>

