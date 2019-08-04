<template>
 <div>
	 <!-- 待优化 2018年4月16日15:33:35 -->
    <Card>
			<p slot="title">
			币币设置
				<Button type="primary" size="small" @click="refreshPageManual">
					<Icon type="refresh"></Icon>刷新
				</Button>
			</p>
      <Row style='padding:0px 8px 8px 8px'>
        <div style="float: right" class="clearfix">
          <Button type="error" size="large"  @click="delcoin">删除交易对</Button>
          <Button type="primary" size="large"  @click="addcoin">添加交易对</Button>
        </div>
      </Row>

      <Modal
          class="auditModel"
          v-model="fixModel"
          title="修改信息"
          @on-ok="confirmFix">
          <ul>
            <li><span><i>*</i> 交易对：</span>
              <p>
                <Input v-model="fixSymbol" disabled></Input>
                <span>{{ }}</span>
              </p>
            </li>

            <li><span><i>*</i> 手续费：</span>
                <p><Input v-model="fee"
                 :class="{'errorFormatBorder': feeClass}" 
                  @on-change="checkFee(fee)"></Input>
                </p>
                <em v-show="feeClass">格式不正确</em>
            </li>

            <li><span><i>*</i> 状态：</span>
              <p>
                <RadioGroup v-model="fixEnable">
                  <Radio label="1"><em>启用</em></Radio>
                  <Radio label="2"><em>禁止</em></Radio>
                </RadioGroup>
              </p>
            </li> 
          </ul>
         
      </Modal>
      <Modal
          class="auditModel"
          v-model="auditModal"
          title="编辑币种"
          @on-ok="confirmAudit">
          <Row>
            <ul>
              <li><span><i>*</i> 交易对：</span>
                <p>
                  <Input v-model="symbol" 
                         :class="{'errorFormatBorder': checkSymbolClass}" 
                         @on-change="checkPair(symbol)"></Input>
                </p>
                <em v-show="checkSymbolClass">格式不正确</em>
              </li>

              <li><span><i>*</i> 交易币种：</span>
                <p><Input v-model="coinSymbol"
                          :class="{'errorFormatBorder': coinSymbolClass}" 
                          @on-change="checkCoinSymbol(coinSymbol)"></Input>
                </p>
                <em v-show="coinSymbolClass">格式不正确</em>
              </li> 

              <li><span><i>*</i> 结算币种：</span>
                <p><Input v-model="baseSymbol"
                          :class="{'errorFormatBorder': baseSymbolClass}" 
                          @on-change="checkbaseSymbol(baseSymbol)"></Input>
                </p>
                 <em v-show="baseSymbolClass">格式不正确</em>
              </li> 

              <li><span><i>*</i> 状态：</span>
                <p>
                  <RadioGroup v-model="enable">
                    <Radio label="1"><em>启用</em></Radio>
                    <Radio label="2"><em>禁止</em></Radio>
                  </RadioGroup>
                </p>
              </li> 

              <li><span><i>*</i> 手续费：</span>
                <p><Input v-model="fee"
                 :class="{'errorFormatBorder': feeClass}" 
                  @on-change="checkFee(fee)"></Input>
                </p>
                <em v-show="feeClass">格式不正确</em>
              </li>

              <li><span><i>*</i> 币种精度：</span>
                <p><Input v-model="coinScale"
                          :class="{'errorFormatBorder': coinScaleClass}" 
                          @on-change="checkCoinScale(coinScale)"></Input>
                </p>
                <em v-show="coinScaleClass">格式不正确</em>
                
              </li>

              <li><span><i>*</i> 基币小数精度：</span>
                <p><Input v-model="baseCoinScale"
                          :class="{'errorFormatBorder': baseCoinScaleClass}" 
                          @on-change="checkbaseCoinScale(baseCoinScale)"></Input>
                </p>
                <em v-show="baseCoinScaleClass">格式不正确</em>
              </li>
            </ul>

          </Row>
      </Modal>
      
      <Table
        border 
        :columns="columns_first" 
        :data="exchange"
        @on-selection-change="selected" 
        :loading="ifLoading">
      </Table>

       <Modal
        v-model="ifDelete"
        title="删除交易对"
        @on-ok="confirmDel"
        @on-cancel="$Message.info('已取消！')">
        <p>是否删除选中的{{ deleteArr.length }}项</p>
       </Modal>

      <Row class="pageWrapper">
        <Page 
        :total="pageNum" 
        class="buttomPage" 
        :current="current"   
        @on-change="changePage" 
        show-elevator></Page>
      </Row>

			 <Modal
					class="auditModel"
					v-model="loginPassModal"
					title="请输入登录密码"
					width="350"
					@on-cancle="loginPW = ''"
					@on-ok="confirmLoginPass">
					<Input v-model="loginPW" type="password" placeholder="请输入登录密码"></Input>
			 </Modal>

    </Card>    
 </div>
</template>

 <script>

import { transactionPair } from '@/caculate/caculate'
import { delBBSetting, addBBSetting, queryBBSetting, fixBBSetting } from '@/service/getData'

export default {
  data() {
    return {
			loginPW: '',
			loginPassModal: false,
      ifLoading: true,
      // currentPageIdx: 1,
      coinScaleClass: false,
      baseSymbolClass: false,
      checkSymbolClass: false,
      coinSymbolClass: false,
      feeClass: false,
      baseCoinScaleClass: false,
      fixSymbol: null,
      fixEnable: 1,
      fixModel: false,
      symbol: null,
      coinSymbol: null,
      baseSymbol: null,
      enable: 1,
      fee: null,
      coinScale: null,
      baseCoinScale: null,

      auditModal: false,
      pageNum: null,
      current: 1,
      deleteArr: false,
      ifDelete: false,
      
      columns_first: [
        {
          type: "selection",
          width: 60,
          algin: 'center'
        },
        {
          title: "序号",
          width: 100,
          type: "index"
        },
        {
          title: "交易对",
          key: "symbol"
        },
        {
          title: "结算币种",
          key: "baseSymbol"
        },
        {
          title: "交易币种",
          key: "coinSymbol"
        },
        {
          title: "手续费",
          key: "fee",
          render: (h, params) => {
            const row = params.row;
            const fee = row.fee * 1000 + "‰";
            return h("span", {}, fee);
          }
        },
        {
          title: "状态",
          key: "enable",
          render: (h, params) => {
            const row = params.row;
            const enable = row.enable == "1" ? "启用" : "禁用";
            return h("span", {}, enable);
          }
        },
        {
          title: "操作",
          key: "xx",
          width: 120,
          render: (h, obj) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "info",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.fixModel = true;
                      this.fixSymbol = obj.row.symbol;
                      this.fee = obj.row.fee;
                      this.fixEnable = String(obj.row.enable);
                    }
                  }
                },
                "修改"
              )
            ]);
          }
        }
      ],
      exchange: []
    };
  },
  methods: {
    checkbaseCoinScale(str) {
      let bol = !(str*1>=0&&String(str).split('.').length===1) 
                || str===null || !str.trim().length;
      this.baseCoinScaleClass =  bol;
      
    },
    checkCoinScale(str) {
      let bol = !(str*1>=0&&String(str).split('.').length===1) 
                || str===null || !str.trim().length;
      this.coinScaleClass =  bol;
    },
    checkFee(str) {
      let re = /\d|\d+.\d+$/g;
      this.feeClass = !re.test(String(str)) || !(str*1>0);
    },
    checkbaseSymbol(str) {
      let re = /[^A-Z]/g;
      this.baseSymbolClass =  re.test(str);
    },
    checkCoinSymbol(str) {
      let re = /[^A-Z]/g;
      this.coinSymbolClass =  re.test(str);

    },
    checkPair(str) {
     let re = /^[A-Z]+\/[A-Z]+$/g;
      this.checkSymbolClass = !re.test(str);
      console.log(this.checkSymbolClass);
      
		},
		confirmLoginPass() {
			  this.feeClass = false;
        let obj ={
          symbol:  this.fixSymbol,
          fee: this.fee,      
					enable: this.fixEnable,
					password: this.loginPW     
				}
				
        fixBBSetting(obj).then( res => {
          if (!res.code) {
						this.$Message.success('修改成功！');
            this.loginPW = '';
            this.current = 1;
            this.refreshdata(1);
          }else this.$Message.error('修改失败！');
        })
		},
    confirmFix() {
			this.loginPassModal =  true;
    },
    confirmAudit() {
      let judgeCondition = this.baseCoinScaleClass || this.coinScaleClass || this.feeClass ||
                  this.baseSymbolClass ||this.coinSymbolClass ||this.checkSymbolClass;
      let judgeEmpty = !this.symbol || !this.coinSymbol || !this.baseSymbol 
                      || !this.fee || !this.coinScale || !this.baseCoinScale;

      if(judgeCondition || judgeEmpty) {
        this.$Message.warning('信息录入不正确！');
      } else {
        let obj= {
          symbol: this.symbol,
          coinSymbol: this.coinSymbol,
          baseSymbol: this.baseSymbol,
          enable: this.enable,
          fee: this.fee,
          coinScale: this.coinScale,
          baseCoinScale: this.baseCoinScale
        }
        addBBSetting(obj).then( res => {
          if(!res.code) {
            this.$Message.success('操作成功！');
            this.current = 1;
            this.refreshdata(1);
          }else this.$Message.error('操作失败！');
  
        })
      }
    },
    selected(selection) {
      this.deleteArr = [];
      selection.forEach(item => {
        this.deleteArr.push(item.symbol)
      });
    },
    confirmDel() {
      delBBSetting({ ids: this.deleteArr }).then( res =>{
        if(!res.code) {
          this.$Message.success('删除成功！');
          this.current = 1;
          this.refreshdata(1);
        }else {
          this.$Message.error('删除失败！');
        }
      })
    },
    refreshPageManual() {
      this.ifLoading = true;
      this.refreshdata(this.current);
      
    },
    changePage(pageIndex) {
      this.ifLoading = true;
      this.current = pageIndex;
      this.refreshdata(pageIndex);
    },
    refreshdata(pageIndex) {
      queryBBSetting({ pageNo: pageIndex, pageSize: 10 }).then( res => {
        this.exchange = res.data.content;
        this.pageNum = res.data.totalElements;
        this.ifLoading = false;
      })
    },
    delcoin() {
      if(!this.deleteArr.length) {
        this.$Message.warning('尚未选取项目！');
      } else  this.ifDelete = true;
    },
    addcoin() {
      this.symbol = null;
      this.coinSymbol = null;
      this.baseSymbol = null;
      this.enable = 1;
      this.fee = null;
      this.coinScale = null;
      this.baseCoinScal = null;
      this.auditModal = true;
    },
  },
  created() {
    this.refreshdata(1);
  }
};
</script>

<style lang="less" scoped>
  .auditModel{
    ul {
      li {
        position: relative;
        margin-bottom: 10px;
        span{
          position: absolute;
          top: 0;
          left: 0;
          display: inline-block;
          width: 95px;
          line-height: 32px;
          text-align: right;
          i{
            font-size: 14px;
            font-weight: 700;
            color: #ec0909;
          }
        }
        em{
          position: absolute;
          top: 0;
          right: 120px;
          line-height: 33px;
          font-style: normal;
          color: #ec0909;
        }
        p{
          padding-left: 100px;
          width: 300px;
          line-height: 32px;
          em{
            position: static;
            color: unset;
          }
        }
      }
    }
  }
</style>