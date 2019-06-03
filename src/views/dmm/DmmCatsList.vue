<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="猫咪编号">
              <a-input placeholder="请输入猫咪编号" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="猫咪名称">
              <a-input placeholder="请输入猫咪名称" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="猫咪公母   0未知  1公  2母">
              <a-input placeholder="请输入猫咪公母   0未知  1公  2母" v-model="queryParam.sex"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="体重(kg)">
              <a-input placeholder="请输入体重(kg)" v-model="queryParam.weight"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="出生日期">
              <a-input placeholder="请输入出生日期" v-model="queryParam.birthday"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('猫咪表')">导出</a-button>
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
    <dmmCats-modal ref="modalForm" @ok="modalFormOk"></dmmCats-modal>
  </a-card>
</template>

<script>
  import DmmCatsModal from './modules/DmmCatsModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmCatsList",
    mixins:[JeecgListMixin],
    components: {
      DmmCatsModal
    },
    data () {
      return {
        description: '猫咪表管理页面',
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
            title: '猫咪编号',
            align:"center",
            dataIndex: 'code'
           },
		   {
            title: '猫咪名称',
            align:"center",
            dataIndex: 'name'
           },
		   {
            title: '公母',
            align:"center",
            dataIndex: 'sex_dictText'
           },
		   {
            title: '体重(kg)',
            align:"center",
            dataIndex: 'weight'
           },
		   {
            title: '出生日期',
            align:"center",
            dataIndex: 'birthday'
           },
		   {
            title: '猫咪描述',
            align:"center",
            dataIndex: 'description'
           },
		   {
            title: '猫咪分类id',
            align:"center",
            dataIndex: 'categoryId'
           },
		   {
            title: '所属会员id',
            align:"center",
            dataIndex: 'belongingUserId'
           },
		   {
            title: '所属供应商id',
            align:"center",
            dataIndex: 'belongingSupplierId'
           },
		   {
            title: '删除标识',
            align:"center",
            dataIndex: 'dataStatus_dictText'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmCats/dmmCats/list",
          delete: "/dmmCats/dmmCats/delete",
          deleteBatch: "/dmmCats/dmmCats/deleteBatch",
          exportXlsUrl: "dmmCats/dmmCats/exportXls",
          importExcelUrl: "dmmCats/dmmCats/importExcel",
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