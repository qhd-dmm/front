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
          label="猫咪编号">
          <a-input placeholder="请输入猫咪编号" v-decorator="['code', validatorRules.code ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="猫咪名称">
          <a-input placeholder="请输入猫咪名称" v-decorator="['name', validatorRules.name ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="猫咪公母   0未知  1公  2母">
          <a-input placeholder="请输入猫咪公母   0未知  1公  2母" v-decorator="['sex', validatorRules.sex ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="体重(kg)">
          <a-input-number v-decorator="[ 'weight', validatorRules.weight ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="出生日期">
          <a-date-picker showTime format='YYYY-MM-DD HH:mm:ss' v-decorator="[ 'birthday', validatorRules.birthday ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="猫咪描述(疫苗情况，血统情况以及其他情况描述 )">
          <a-input placeholder="请输入猫咪描述(疫苗情况，血统情况以及其他情况描述 )" v-decorator="['description', validatorRules.description ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="猫咪分类id">
          <a-input-number v-decorator="[ 'categoryId', validatorRules.categoryId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="所属会员id">
          <a-input-number v-decorator="[ 'belongingUserId', validatorRules.belongingUserId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="所属供应商id">
          <a-input-number v-decorator="[ 'belongingSupplierId', validatorRules.belongingSupplierId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="删除标识  -1删除   1未删除">
          <a-input placeholder="请输入删除标识  -1删除   1未删除" v-decorator="['dataStatus', validatorRules.dataStatus ]" />
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
    name: "DmmCatsModal",
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
        code:{rules: [{ required: true, message: '请输入猫咪编号!' }]},
        name:{rules: [{ required: true, message: '请输入猫咪名称!' }]},
        sex:{rules: [{ required: true, message: '请输入猫咪公母   0未知  1公  2母!' }]},
        weight:{rules: [{ required: true, message: '请输入体重(kg)!' }]},
        birthday:{rules: [{ required: true, message: '请输入出生日期!' }]},
        description:{rules: [{ required: true, message: '请输入猫咪描述(疫苗情况，血统情况以及其他情况描述 )!' }]},
        categoryId:{rules: [{ required: true, message: '请输入猫咪分类id!' }]},
        belongingUserId:{rules: [{ required: true, message: '请输入所属会员id!' }]},
        belongingSupplierId:{rules: [{ required: true, message: '请输入所属供应商id!' }]},
        dataStatus:{rules: [{ required: true, message: '请输入删除标识  -1删除   1未删除!' }]},
        },
        url: {
          add: "/dmmCats/dmmCats/add",
          edit: "/dmmCats/dmmCats/edit",
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
          this.form.setFieldsValue(pick(this.model,'code','name','sex','weight','description','categoryId','belongingUserId','belongingSupplierId','dataStatus'))
		  //时间格式化
          this.form.setFieldsValue({birthday:this.model.birthday?moment(this.model.birthday):null})
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
            formData.birthday = formData.birthday?formData.birthday.format('YYYY-MM-DD HH:mm:ss'):null;
            
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