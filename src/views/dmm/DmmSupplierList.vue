<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="供应商编号">
              <a-input placeholder="请输入供应商编号" v-model="queryParam.supplierCode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="供应商名称">
              <a-input placeholder="请输入供应商名称" v-model="queryParam.supplierName"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="供应商手机">
              <a-input placeholder="请输入供应商手机" v-model="queryParam.supplierPhone"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="供应商地址">
              <a-input placeholder="请输入供应商地址" v-model="queryParam.address"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="供应商状态    0停用  1启用">
              <a-input placeholder="请输入供应商状态    0停用  1启用" v-model="queryParam.supplierStatus"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('供应商表')">导出</a-button>
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
    <dmmSupplier-modal ref="modalForm" @ok="modalFormOk"></dmmSupplier-modal>
  </a-card>
</template>

<script>
  import DmmSupplierModal from './modules/DmmSupplierModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmSupplierList",
    mixins:[JeecgListMixin],
    components: {
      DmmSupplierModal
    },
    data () {
      return {
        description: '供应商表管理页面',
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
            title: '供应商编号',
            align:"center",
            dataIndex: 'supplierCode'
           },
		   {
            title: '供应商名称',
            align:"center",
            dataIndex: 'supplierName'
           },
		   {
            title: '供应商手机',
            align:"center",
            dataIndex: 'supplierPhone'
           },
		   {
            title: '供应商地址',
            align:"center",
            dataIndex: 'address'
           },
		   {
            title: '供应商状态    0停用  1启用',
            align:"center",
            dataIndex: 'supplierStatus'
           },
		   {
            title: '供应商登陆名',
            align:"center",
            dataIndex: 'loginName'
           },
		   {
            title: '供应商登陆密码',
            align:"center",
            dataIndex: 'loginPwd'
           },
		   {
            title: '供应商类型   1.活体 2.食品用品  3服务商',
            align:"center",
            dataIndex: 'supplierType'
           },
		   {
            title: '是否合格   0不合格   1合格',
            align:"center",
            dataIndex: 'qualifiedStatus'
           },
		   {
            title: '删除标志    -1已删除   1未删除',
            align:"center",
            dataIndex: 'dateStatus'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmSupplier/dmmSupplier/list",
          delete: "/dmmSupplier/dmmSupplier/delete",
          deleteBatch: "/dmmSupplier/dmmSupplier/deleteBatch",
          exportXlsUrl: "dmmSupplier/dmmSupplier/exportXls",
          importExcelUrl: "dmmSupplier/dmmSupplier/importExcel",
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