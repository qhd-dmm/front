<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)">
              <a-input placeholder="请输入订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)" v-model="queryParam.orderStatus"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="收货方式	0:送货上门 1:自提">
              <a-input placeholder="请输入收货方式	0:送货上门 1:自提" v-model="queryParam.deliverType"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="客户id">
              <a-input placeholder="请输入客户id" v-model="queryParam.customId"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="清单总金额">
              <a-input placeholder="请输入清单总金额" v-model="queryParam.billMoney"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="运费">
              <a-input placeholder="请输入运费" v-model="queryParam.deliverMoney"></a-input>
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
            title: '订单状态(1.待付款  2.待发货  3.待收货   4.交易完成  5.交易关闭)',
            align:"center",
            dataIndex: 'orderStatus'
           },
		   {
            title: '收货方式	0:送货上门 1:自提',
            align:"center",
            dataIndex: 'deliverType'
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
            title: '订单总金额  包括运费',
            align:"center",
            dataIndex: 'totalMoney'
           },
		   {
            title: '实际订单总金额 进行各种折扣之后的金额',
            align:"center",
            dataIndex: 'realTotalMoney'
           },
		   {
            title: '支付方式  0:货到付款 1:先款后货',
            align:"center",
            dataIndex: 'payType'
           },
		   {
            title: '支付来源  1:现金，2：支付宝  3微信',
            align:"center",
            dataIndex: 'payFrom'
           },
		   {
            title: '是否支付定金 0:未支付 1:已支付',
            align:"center",
            dataIndex: 'isPayFront'
           },
		   {
            title: '是否支付尾款	0:未支付 1:已支付',
            align:"center",
            dataIndex: 'isPayAll'
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
            title: '是否需要发票   1:需要 0:不需要',
            align:"center",
            dataIndex: 'isInvoice'
           },
		   {
            title: '发票抬头',
            align:"center",
            dataIndex: 'invoiceClient'
           },
		   {
            title: '订单来源	0:门店 1:微信',
            align:"center",
            dataIndex: 'orderSrc'
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
            title: '订单描述(针对于发猫前的进展信息)',
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
            title: '0.未发生退款       1.退款中       2退款完成',
            align:"center",
            dataIndex: 'refundStatus'
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
            title: '是否包含套餐 1是  -1不是',
            align:"center",
            dataIndex: 'ifSetMeal'
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