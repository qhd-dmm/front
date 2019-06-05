<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">

            <a-form-item label="订单状态">
              <j-dict-select-tag v-model="queryParam.orderStatus" placeholder="请选择订单状态" dictCode="order_status"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="收货方式">
              <j-dict-select-tag v-model="queryParam.deliverType" placeholder="请选择收获方式" dictCode="deliver_type"/>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
          <a-col :md="6" :sm="8">
            <a-form-item label="支付方式">
              <j-dict-select-tag v-model="queryParam.payType" placeholder="请选择支付方式" dictCode="pay_type"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="支付来源">
              <j-dict-select-tag v-model="queryParam.payFrom" placeholder="请选择支付来源" dictCode="pay_from"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="是否支付定金">
              <j-dict-select-tag v-model="queryParam.isPayFront" placeholder="请选择是否支付定金" dictCode="is_pay_front"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="是否支付尾款">
              <j-dict-select-tag v-model="queryParam.isPayAll" placeholder="请选择是否支付尾款" dictCode="is_pay_all"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="订单来源">
              <j-dict-select-tag v-model="queryParam.orderSrc" placeholder="请选择订单来源" dictCode="order_src"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="退款状态">
              <j-dict-select-tag v-model="queryParam.refundStatus" placeholder="请选择退款状态" dictCode="refund_status"/>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="所属员工id">
              <j-dict-select-tag v-model="queryParam.refundStatus" placeholder="请选择退款状态" dictCode="refund_status"/>
            </a-form-item>
          </a-col>
        </template>
          <a-col :md="6" :sm="8" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('订单主表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <dmmOrder-modal ref="modalForm" @ok="modalFormOk"></dmmOrder-modal>
  </a-card>
</template>

<script>
  import DmmOrderModal from './modules/DmmOrderModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmOrderList",
    mixins:[JeecgListMixin],
    components: {
      DmmOrderModal
    },
    data () {
      return {
        description: '订单主表管理页面',
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
           },
		   {
            title: '订单状态',
            align:"center",
            dataIndex: 'orderStatus_dictText'
           },
		   {
            title: '收货方式',
            align:"center",
            dataIndex: 'deliverType_dictText'
           },
		   {
            title: '客户id',
            align:"center",
            dataIndex: 'customId'
           },
		   {
            title: '清单总金额',
            align:"center",
            dataIndex: 'billMoney'
           },
		   {
            title: '运费',
            align:"center",
            dataIndex: 'deliverMoney'
           },
		   {
            title: '订单总金额',
            align:"center",
            dataIndex: 'totalMoney'
           },
		   {
            title: '折后总金额',
            align:"center",
            dataIndex: 'realTotalMoney_dictText'
           },
		   {
            title: '支付方式',
            align:"center",
            dataIndex: 'payType_dictText'
           },
		   {
            title: '支付来源',
            align:"center",
            dataIndex: 'payFrom_dictText'
           },
		   {
            title: '是否支付定金',
            align:"center",
            dataIndex: 'isPayFront_dictText'
           },
		   {
            title: '是否支付尾款',
            align:"center",
            dataIndex: 'isPayAll_dictText'
           },
		   {
            title: '收货人姓名',
            align:"center",
            dataIndex: 'userName'
           },
		   {
            title: '收件人地址',
            align:"center",
            dataIndex: 'userAddress'
           },
		   {
            title: '收件人手机',
            align:"center",
            dataIndex: 'userPhone'
           },
		   {
            title: '所得积分',
            align:"center",
            dataIndex: 'orderScore'
           },
		   {
            title: '是否需要发票',
            align:"center",
            dataIndex: 'isInvoice_dictText'
           },
		   {
            title: '发票抬头',
            align:"center",
            dataIndex: 'invoiceClient'
           },
		   {
            title: '订单来源',
            align:"center",
            dataIndex: 'orderSrc_dictText'
           },
		   {
            title: '收货时间',
            align:"center",
            dataIndex: 'receiveTime'
           },
		   {
            title: '发货时间',
            align:"center",
            dataIndex: 'deliveryTime'
           },
		   {
            title: '订单描述',
            align:"center",
            dataIndex: 'orderDescription'
           },
		   {
            title: '订单备注',
            align:"center",
            dataIndex: 'orderRemarks'
           },
		   {
            title: '所属员工id',
            align:"center",
            dataIndex: 'staffId'
           },
		   {
            title: '退款状态',
            align:"center",
            dataIndex: 'refundStatus_dictText'
           },
		   {
            title: '快递公司ID',
            align:"center",
            dataIndex: 'expressId'
           },
		   {
            title: '快递号',
            align:"center",
            dataIndex: 'expressNo'
           },
		   {
            title: '是否包含套餐',
            align:"center",
            dataIndex: 'ifSetMeal_dictText'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmOrder/dmmOrder/list",
          delete: "/dmmOrder/dmmOrder/delete",
          deleteBatch: "/dmmOrder/dmmOrder/deleteBatch",
          exportXlsUrl: "dmmOrder/dmmOrder/exportXls",
          importExcelUrl: "dmmOrder/dmmOrder/importExcel",
       },
    }
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    }
  },
    methods: {
     
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>