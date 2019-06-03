<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购类型">
          <j-dict-select-tag  :triggerChange="true" dictCode="purchase_type"  v-decorator="['purchaseType', validatorRules.purchaseType]" placeholder="请选择采购类型">
          </j-dict-select-tag>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购状态">
          <j-dict-select-tag  :triggerChange="true" dictCode="purchase_status"  v-decorator="['purchaseStatus', validatorRules.purchaseStatus]" placeholder="请选择采购状态">
          </j-dict-select-tag>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="采购单合计价格">
          <a-input-number v-decorator="[ 'totalPrice', validatorRules.totalPrice ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="申请时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'applyTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label=" 到货时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'receiveTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="核销时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'closeAcountTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="期望到货时间">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'expectTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="申请人">
          <a-input-number v-decorator="[ 'applyStaffId', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="执行人">
          <a-input-number v-decorator="[ 'staffId', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="进货备注">
          <a-input placeholder="请输入进货备注" v-decorator="['purchaseMarks', {}]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "DmmPurchaseModal",
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
        purchaseType:{rules: [{ required: true, message: '请输入采购类型(1.订单采购  2.公司日常采购   3.固定资产采购)!' }]},
        purchaseStatus:{rules: [{ required: true, message: '请输入采购状态(1.已申请   2.已到货    3已核销)!' }]},
        totalPrice:{rules: [{ required: true, message: '请输入采购单合计价格!' }]},
        },
        url: {
          add: "/dmmPurchase/dmmPurchase/add",
          edit: "/dmmPurchase/dmmPurchase/edit",
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
          this.form.setFieldsValue(pick(this.model,'purchaseType','purchaseStatus','totalPrice','applyStaffId','staffId','purchaseMarks'))
		  //时间格式化
          this.form.setFieldsValue({applyTime:this.model.applyTime?moment(this.model.applyTime):null})
          this.form.setFieldsValue({receiveTime:this.model.receiveTime?moment(this.model.receiveTime):null})
          this.form.setFieldsValue({closeAcountTime:this.model.closeAcountTime?moment(this.model.closeAcountTime):null})
          this.form.setFieldsValue({expectTime:this.model.expectTime?moment(this.model.expectTime):null})
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
            formData.applyTime = formData.applyTime?formData.applyTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.receiveTime = formData.receiveTime?formData.receiveTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.closeAcountTime = formData.closeAcountTime?formData.closeAcountTime.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.expectTime = formData.expectTime?formData.expectTime.format('YYYY-MM-DD HH:mm:ss'):null;
            
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

</style>