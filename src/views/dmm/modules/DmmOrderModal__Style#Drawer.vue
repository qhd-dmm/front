<template>
  <a-drawer
      :title="title"
      :width="800"
      placement="right"
      :closable="false"
      @close="close"
      :visible="visible"
  >

    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)">
          <a-input placeholder="请输入订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)" v-decorator="['orderStatus', validatorRules.orderStatus ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="收货方式	0:送货上门 1:自提">
          <a-input placeholder="请输入收货方式	0:送货上门 1:自提" v-decorator="['deliverType', validatorRules.deliverType ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="客户id">
          <a-input-number v-decorator="[ 'customId', validatorRules.customId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="清单总金额">
          <a-input-number v-decorator="[ 'billMoney', validatorRules.billMoney ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="运费">
          <a-input-number v-decorator="[ 'deliverMoney', validatorRules.deliverMoney ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="订单总金额  包括运费">
          <a-input-number v-decorator="[ 'totalMoney', validatorRules.totalMoney ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="实际订单总金额 进行各种折扣之后的金额">
          <a-input-number v-decorator="[ 'realTotalMoney', validatorRules.realTotalMoney ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="支付方式  0:货到付款 1:先款后货">
          <a-input placeholder="请输入支付方式  0:货到付款 1:先款后货" v-decorator="['payType', validatorRules.payType ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="支付来源  1:现金，2：支付宝  3微信">
          <a-input placeholder="请输入支付来源  1:现金，2：支付宝  3微信" v-decorator="['payFrom', validatorRules.payFrom ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否支付定金 0:未支付 1:已支付">
          <a-input placeholder="请输入是否支付定金 0:未支付 1:已支付" v-decorator="['isPayFront', validatorRules.isPayFront ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否支付尾款	0:未支付 1:已支付">
          <a-input placeholder="请输入是否支付尾款	0:未支付 1:已支付" v-decorator="['isPayAll', validatorRules.isPayAll ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="收货人姓名">
          <a-input placeholder="请输入收货人姓名" v-decorator="['userName', validatorRules.userName ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="收件人地址">
          <a-input placeholder="请输入收件人地址" v-decorator="['userAddress', validatorRules.userAddress ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="收件人手机">
          <a-input placeholder="请输入收件人手机" v-decorator="['userPhone', validatorRules.userPhone ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="所得积分">
          <a-input-number v-decorator="[ 'orderScore', validatorRules.orderScore ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否需要发票   1:需要 0:不需要">
          <a-input placeholder="请输入是否需要发票   1:需要 0:不需要" v-decorator="['isInvoice', validatorRules.isInvoice ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="发票抬头">
          <a-input placeholder="请输入发票抬头" v-decorator="['invoiceClient', validatorRules.invoiceClient ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="订单来源	0:门店 1:微信">
          <a-input placeholder="请输入订单来源	0:门店 1:微信" v-decorator="['orderSrc', validatorRules.orderSrc ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="收货时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'receiveTime', validatorRules.receiveTime ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="发货时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'deliveryTime', validatorRules.deliveryTime ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="订单描述(针对于发猫前的进展信息)">
          <a-input placeholder="请输入订单描述(针对于发猫前的进展信息)" v-decorator="['orderDescription', validatorRules.orderDescription ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="订单备注">
          <a-input placeholder="请输入订单备注" v-decorator="['orderRemarks', validatorRules.orderRemarks ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="所属员工id">
          <a-input-number v-decorator="[ 'staffId', validatorRules.staffId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="0.未发生退款       1.退款中       2退款完成">
          <a-input placeholder="请输入0.未发生退款       1.退款中       2退款完成" v-decorator="['refundStatus', validatorRules.refundStatus ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="快递公司ID">
          <a-input placeholder="请输入快递公司ID" v-decorator="['expressId', validatorRules.expressId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="快递号">
          <a-input placeholder="请输入快递号" v-decorator="['expressNo', validatorRules.expressNo ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="是否包含套餐 1是  -1不是">
          <a-input placeholder="请输入是否包含套餐 1是  -1不是" v-decorator="['ifSetMeal', {}]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "DmmOrderModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        orderStatus:{rules: [{ required: true, message: '请输入订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)!' }]},
        deliverType:{rules: [{ required: true, message: '请输入收货方式	0:送货上门 1:自提!' }]},
        customId:{rules: [{ required: true, message: '请输入客户id!' }]},
        billMoney:{rules: [{ required: true, message: '请输入清单总金额!' }]},
        deliverMoney:{rules: [{ required: true, message: '请输入运费!' }]},
        totalMoney:{rules: [{ required: true, message: '请输入订单总金额  包括运费!' }]},
        realTotalMoney:{rules: [{ required: true, message: '请输入实际订单总金额 进行各种折扣之后的金额!' }]},
        payType:{rules: [{ required: true, message: '请输入支付方式  0:货到付款 1:先款后货!' }]},
        payFrom:{rules: [{ required: true, message: '请输入支付来源  1:现金，2：支付宝  3微信!' }]},
        isPayFront:{rules: [{ required: true, message: '请输入是否支付定金 0:未支付 1:已支付!' }]},
        isPayAll:{rules: [{ required: true, message: '请输入是否支付尾款	0:未支付 1:已支付!' }]},
        userName:{rules: [{ required: true, message: '请输入收货人姓名!' }]},
        userAddress:{rules: [{ required: true, message: '请输入收件人地址!' }]},
        userPhone:{rules: [{ required: true, message: '请输入收件人手机!' }]},
        orderScore:{rules: [{ required: true, message: '请输入所得积分!' }]},
        isInvoice:{rules: [{ required: true, message: '请输入是否需要发票   1:需要 0:不需要!' }]},
        invoiceClient:{rules: [{ required: true, message: '请输入发票抬头!' }]},
        orderSrc:{rules: [{ required: true, message: '请输入订单来源	0:门店 1:微信!' }]},
        receiveTime:{rules: [{ required: true, message: '请输入收货时间!' }]},
        deliveryTime:{rules: [{ required: true, message: '请输入发货时间!' }]},
        orderDescription:{rules: [{ required: true, message: '请输入订单描述(针对于发猫前的进展信息)!' }]},
        orderRemarks:{rules: [{ required: true, message: '请输入订单备注!' }]},
        staffId:{rules: [{ required: true, message: '请输入所属员工id!' }]},
        refundStatus:{rules: [{ required: true, message: '请输入0.未发生退款       1.退款中       2退款完成!' }]},
        expressId:{rules: [{ required: true, message: '请输入快递公司ID!' }]},
        expressNo:{rules: [{ required: true, message: '请输入快递号!' }]},
        },
        url: {
          add: "/dmmOrder/dmmOrder/add",
          edit: "/dmmOrder/dmmOrder/edit",
        },
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'orderStatus','deliverType','customId','billMoney','deliverMoney','totalMoney','realTotalMoney','payType','payFrom','isPayFront','isPayAll','userName','userAddress','userPhone','orderScore','isInvoice','invoiceClient','orderSrc','orderDescription','orderRemarks','staffId','refundStatus','expressId','expressNo','ifSetMeal'))
		  //时间格式化
          this.form.setFieldsValue({receiveTime:this.model.receiveTime?moment(this.model.receiveTime):null})
          this.form.setFieldsValue({deliveryTime:this.model.deliveryTime?moment(this.model.deliveryTime):null})
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.receiveTime = formData.receiveTime?formData.receiveTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.deliveryTime = formData.deliveryTime?formData.deliveryTime.format('YYYY-MM-DD HH:mm:ss'):null;
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },


    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
</style>