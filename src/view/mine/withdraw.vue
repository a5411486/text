<template>
  <div>
    <van-nav-bar left-arrow @click-left="_routerBack" :title="$route.query.name" fixed :z-index="10"></van-nav-bar>
    <div style="margin-top:50px;display:flex">
    </div>
    <div class v-if="this.judge==3">
      <van-list v-model="isUpLoading" :finished="upFinished" finished-text="没有更多了" @load="onLoad">
        <van-cell v-for="(item,index) in lineUp" :key="item.id">
          <template slot-scope="scpoed">
            <div class="lineColor" :style="id==item.userId? backgroundColor:''">

              <div style="flex:1;margin-left: 20px;">
                <div>
                  　　排队: {{index+1}}
                </div>
                <div>
                  共享积分: {{item.shareIntegral}}
                </div>
                <div style="text-align:left">
                  消费金额: {{item.amount}}
                </div>
              </div>
              <div style="flex:1">
                <div style="color:#ffffff">
                  　 -
                </div>
                <div style="text-align:left">
                  　用户名: {{item.userName}}
                </div>

                <div style="text-align:left">
                  消费时间: {{item.consumptionTime}}
                </div>
                <div style="text-align:right">
                </div>
              </div>
            </div>
            <div v-if="item.integralState==1" style="text-align:center;padding-top: 10px;"
              :style="id==item.userId? backgroundColor:''">
              <van-button type="primary" size="small" @click="onClickButtonSubmit(item.amount,item.id)"
                v-if="id==item.userId">提现
              </van-button>
            </div>
          </template>
        </van-cell>
      </van-list>
    </div>
    <div v-if="this.judge==1||this.judge==2">
      <van-field v-model="name" disabled clearable label="姓名" placeholder="请输入项目的费用" />
      <van-field v-model="Alipay" disabled clearable label="支付宝账号" placeholder="请输入项目的费用" />
      <van-field v-model="money" disabled clearable label="可提现金额" />
      <van-field v-if="this.judge==1" v-model="Tqu" placeholder="请输入提现金额" clearable label="提现金" />
      <van-field v-if="this.judge==2" v-model="Tqu" clearable label="提现金" placeholder="请选择提现金额" @focus="start" />
      <van-popup v-model="show" position="bottom">
        <van-picker show-toolbar title="请选择提现金额" v-model="amount" :columns="columns" @cancel="cancel"
          @confirm="confirm" />
      </van-popup>
    </div>
    <div style="text-align:center;margin:20px;">
      <van-button v-if="this.judge==1||this.judge==2" type="primary" size="large" @click="onClickButtonSubmit"><span
          v-if="this.judge==1">确认提现</span><span v-if="this.judge==2">确认</span></van-button>
    </div>

  </div>
</template>
<script>
  // import { DropdownMenu, DropdownItem } from 'vant';

  import {
    Popup
  } from 'vant';
  import {
    Picker
  } from 'vant';
  export default {

    data() {
      return {
        backgroundColor: 'background: antiquewhite;',
        columns: [],
        list: [],
        isUpLoading: false,
        upFinished: false,
        judge: 1,
        name: '',
        Alipay: '',
        money: this.$route.query.money + '.00',
        Tqu: '',
        userMony: '',
        shopping: '',
        lineUp: [],
        page: 1,
        pageSize: 100,
        buttonShare: '',
        show: false,
        listSelect: [],
        id: '',
        amount: ''

      }
    },
    methods: {
      // confirm() {
      //   console.log(1);
      //   this.show = false;

      // },
      cancel() {
        this.show = false;
      },
      start() {
        this.show = true;
      },
      onLoad() {
        this.pageSize++
        this.integration()
      },
      onClickButtonSubmit(value) {
        let id
        this.listSelect.map(item => {
          if (this.Tqu == item.amount) {
            id = item.id
          }
        })



        if ((this.Tqu && this.Alipay) ||( value&& this.Alipay)) {

          let param = {
            userId: JSON.parse(localStorage.getItem("user")).id,
            amount:this.judge==3? value:this.Tqu,
            type: this.judge,
            consumptionId: id,
            alipayUser:this.Alipay
          }
          if (!isNaN(value)) {
            param.amount = value
          }
          this.$http.post(`api/lyRecord/save`, param).then(res => {
            console.log(res.data.code);
            if (res.data.code == 200) {
              this._showLoading();
              setTimeout(() => {
                this._showToast('提现申请成功');
                this.$router.go(-1)
              }, 1000);

            }
            if (res.data.code == 100) {
              this._showToast(res.data.message);
              // Toast(res.data.message);
            }
          }).catch(err => {
            this._dismissLoading();
            this._showToast('系统错误');
          });

        } else {
          console.log(this.Alipay);
          if (this.Alipay) {
            this._showToast('请选择金额');

          } else {
            this._showToast('请到设置里面设置支付宝账号');

          }
        }
      },
      selectWithdraw() {
        let param = {
          userId: JSON.parse(localStorage.getItem('user')).id
        }
        this.$http.post(`/login/lyConsumption/selectOne`, param).then(res => {
          console.log(res);
          this.listSelect = res.data.data
          res.data.data.map(item => {
            this.columns.push(item.amount)
          })
        })
      },
      confirm(value) {
        console.log(value);
        this.show = false
        this.Tqu = value
      },
      integration() {
        console.log(1);
        const _self = this
        let param = {
          page: this.page,
          size: this.pageSize,
          state: 1,
          recordState: 0
        }
        this.$http.post(`/login/lyConsumption/selectALL`, param).then(res => {
          if (res.data.code === 200) {
            const rows = res.data.data.list
            if (rows == null || rows.length === 0) {
              // 加载结束
              _self.upFinished = true
              return
            }
            if (rows.length < this.pageSize) {
              // 最后一页不足10条的情况
              _self.upFinished = true
            }
            // 处理数据
            if (_self.pageIndex === 1) {
              _self.lineUp = rows
            } else {
              _self.lineUp = _self.lineUp.concat(rows)
            }
          }

        }).catch(error => {
          this.$message({
            showClose: true,
            message: '获取资源列表异常{' + error + '}',
            type: 'error'
          })
        }).finally(() => {
          _self.isDownLoading = false
          _self.isUpLoading = false
        })

      },
    },
    mounted() {
      this.selectWithdraw()
      this.Alipay = JSON.parse(localStorage.getItem("user")).alipayUser
      this.name = JSON.parse(localStorage.getItem("user")).userName
      this.id = JSON.parse(localStorage.getItem("user")).id
      if (this.$route.query.name == "奖励积分") {
        this.judge = 1
      }
      if (this.$route.query.name == "优享积分") {
        this.judge = 2
      }
      if (this.$route.query.name == "共享积分") {
        this.judge = 3
      }
    }

  }

</script>
<style lang="scss" scoped>
  .van-cell__value {
    border: 1px solid #e4e4e4;
    box-shadow: 3px 3px 3px 0px #ccc;
    margin: 0px 5px;
  }

  .lineColor {
    display: flex;
    font-size: 12px;
  }

</style>
