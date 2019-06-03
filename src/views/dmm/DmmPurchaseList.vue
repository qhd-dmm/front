<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="采购类型(1.订单采购  2.公司日常采购   3.固定资产采购)">
              <a-input placeholder="请输入采购类型(1.订单采购  2.公司日常采购   3.固定资产采购)" v-model="queryParam.purchaseType"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="采购状态(1.已申请   2.已到货    3已核销)">
              <a-input placeholder="请输入采购状态(1.已申请   2.已到货    3已核销)" v-model="queryParam.purchaseStatus"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="采购单合计价格">
              <a-input placeholder="请输入采购单合计价格" v-model="queryParam.totalPrice"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="申请时间">
              <a-input placeholder="请输入申请时间" v-model="queryParam.applyTime"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label=" 到货时间">
              <a-input placeholder="请输入 到货时间" v-model="queryParam.receiveTime"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('采购主表')">导出</a-button>
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
    <dmmPurchase-modal ref="modalForm" @ok="modalFormOk"></dmmPurchase-modal>
  </a-card>
</template>

<script>
  import DmmPurchaseModal from './modules/DmmPurchaseModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmPurchaseList",
    mixins:[JeecgListMixin],
    components: {
      DmmPurchaseModal
    },
    data () {
      return {
        description: '采购主表管理页面',
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
            title: '采购类型',
            align:"center",
            dataIndex: 'purchaseType_dictText'
           },
		   {
            title: '采购状态',
            align:"center",
            dataIndex: 'purchaseStatus_dictText'
           },
		   {
            title: '采购单合计价格',
            align:"center",
            dataIndex: 'totalPrice'
           },
		   {
            title: '申请时间',
            align:"center",
            dataIndex: 'applyTime'
           },
		   {
            title: ' 到货时间',
            align:"center",
            dataIndex: 'receiveTime'
           },
		   {
            title: '核销时间',
            align:"center",
            dataIndex: 'closeAcountTime'
           },
		   {
            title: '期望到货时间',
            align:"center",
            dataIndex: 'expectTime'
           },
		   {
            title: '申请人',
            align:"center",
            dataIndex: 'applyStaffId'
           },
		   {
            title: '执行人',
            align:"center",
            dataIndex: 'staffId'
           },
		   {
            title: '进货备注',
            align:"center",
            dataIndex: 'purchaseMarks'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmPurchase/dmmPurchase/list",
          delete: "/dmmPurchase/dmmPurchase/delete",
          deleteBatch: "/dmmPurchase/dmmPurchase/deleteBatch",
          exportXlsUrl: "dmmPurchase/dmmPurchase/exportXls",
          importExcelUrl: "dmmPurchase/dmmPurchase/importExcel",
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