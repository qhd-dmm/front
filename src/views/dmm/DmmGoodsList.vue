<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="商品编号">
              <a-input placeholder="请输入商品编号" v-model="queryParam.goodsCode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="商品货号">
              <a-input placeholder="请输入商品货号" v-model="queryParam.productNo"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="商品名称">
              <a-input placeholder="请输入商品名称" v-model="queryParam.goodsName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="商品图片地址">
              <a-input placeholder="请输入商品图片地址" v-model="queryParam.goodsImg"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="市场价格">
              <a-input placeholder="请输入市场价格" v-model="queryParam.marketPrice"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('商品表')">导出</a-button>
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
    <dmmGoods-modal ref="modalForm" @ok="modalFormOk"></dmmGoods-modal>
  </a-card>
</template>

<script>
  import DmmGoodsModal from './modules/DmmGoodsModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmGoodsList",
    mixins:[JeecgListMixin],
    components: {
      DmmGoodsModal
    },
    data () {
      return {
        description: '商品表管理页面',
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
            title: '商品编号',
            align:"center",
            dataIndex: 'goodsCode'
           },
		   {
            title: '商品货号',
            align:"center",
            dataIndex: 'productNo'
           },
		   {
            title: '商品名称',
            align:"center",
            dataIndex: 'goodsName'
           },
		   {
            title: '商品图片地址',
            align:"center",
            dataIndex: 'goodsImg'
           },
		   {
            title: '市场价格',
            align:"center",
            dataIndex: 'marketPrice'
           },
		   {
            title: '门店价',
            align:"center",
            dataIndex: 'shopPrice'
           },
		   {
            title: '预警库存',
            align:"center",
            dataIndex: 'warnStock'
           },
		   {
            title: '商品总库存',
            align:"center",
            dataIndex: 'goodsStock'
           },
		   {
            title: '单位表id',
            align:"center",
            dataIndex: 'goodsUnit'
           },
		   {
            title: '促销信息',
            align:"center",
            dataIndex: 'goodsTips'
           },
		   {
            title: '是否上架	0:不上架 1:上架',
            align:"center",
            dataIndex: 'isSale'
           },
		   {
            title: '是否热销产品	0:否 1:是',
            align:"center",
            dataIndex: 'isHot'
           },
		   {
            title: '商品分类ID路径',
            align:"center",
            dataIndex: 'goodsCatPath'
           },
		   {
            title: '最后一级商品分类ID	',
            align:"center",
            dataIndex: 'goodsCatId'
           },
		   {
            title: '品牌Id',
            align:"center",
            dataIndex: 'brandId'
           },
		   {
            title: '商品描述	',
            align:"center",
            dataIndex: 'goodsDesc'
           },
		   {
            title: '商品状态	-1:违规 0:未审核 1:已审核',
            align:"center",
            dataIndex: 'goodsStatus'
           },
		   {
            title: '总销售量',
            align:"center",
            dataIndex: 'saleNum'
           },
		   {
            title: '上架时间',
            align:"center",
            dataIndex: 'saleTime'
           },
		   {
            title: '删除标志	-1:删除 1:有效',
            align:"center",
            dataIndex: 'dataStatus'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmGoods/dmmGoods/list",
          delete: "/dmmGoods/dmmGoods/delete",
          deleteBatch: "/dmmGoods/dmmGoods/deleteBatch",
          exportXlsUrl: "dmmGoods/dmmGoods/exportXls",
          importExcelUrl: "dmmGoods/dmmGoods/importExcel",
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