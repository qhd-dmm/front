<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="登陆名">
              <a-input placeholder="请输入登陆名" v-model="queryParam.loginName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="密码">
              <a-input placeholder="请输入密码" v-model="queryParam.loginPwd"></a-input>
            </a-form-item>
          </a-col>
        <template v-if="toggleSearchStatus">
        <a-col :md="6" :sm="8">
            <a-form-item label="0保密  1男 2女">
              <a-input placeholder="请输入0保密  1男 2女" v-model="queryParam.userSex"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="昵称">
              <a-input placeholder="请输入昵称" v-model="queryParam.nickName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="真实姓名">
              <a-input placeholder="请输入真实姓名" v-model="queryParam.trueName"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('会员表')">导出</a-button>
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
    <dmmUser-modal ref="modalForm" @ok="modalFormOk"></dmmUser-modal>
  </a-card>
</template>

<script>
  import DmmUserModal from './modules/DmmUserModal'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'

  export default {
    name: "DmmUserList",
    mixins:[JeecgListMixin],
    components: {
      DmmUserModal
    },
    data () {
      return {
        description: '会员表管理页面',
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
            title: '登陆名',
            align:"center",
            dataIndex: 'loginName'
           },
		   {
            title: '密码',
            align:"center",
            dataIndex: 'loginPwd'
           },
		   {
            title: '0保密  1男 2女',
            align:"center",
            dataIndex: 'userSex'
           },
		   {
            title: '昵称',
            align:"center",
            dataIndex: 'nickName'
           },
		   {
            title: '真实姓名',
            align:"center",
            dataIndex: 'trueName'
           },
		   {
            title: '出生日期',
            align:"center",
            dataIndex: 'brithday'
           },
		   {
            title: '用户头像地址',
            align:"center",
            dataIndex: 'userPhoto'
           },
		   {
            title: '用户手机',
            align:"center",
            dataIndex: 'userPhone'
           },
		   {
            title: '用户积分',
            align:"center",
            dataIndex: 'userScore'
           },
		   {
            title: '用户剩余积分',
            align:"center",
            dataIndex: 'userSurplusScore'
           },
		   {
            title: '用户编号',
            align:"center",
            dataIndex: 'userCode'
           },
		   {
            title: '收入水平(1:0-2000    2:2000-5000   3:5000-1w   4:1w+)',
            align:"center",
            dataIndex: 'income'
           },
		   {
            title: '职业 1.学生 2白领 3个体',
            align:"center",
            dataIndex: 'profession'
           },
		   {
            title: '推荐人id',
            align:"center",
            dataIndex: 'refereeUser'
           },
		   {
            title: '会员等级id',
            align:"center",
            dataIndex: 'rankId'
           },
		   {
            title: '用户账号状态 0:停用 1:启用',
            align:"center",
            dataIndex: 'userStatus'
           },
		   {
            title: '	-1:删除 1:有效',
            align:"center",
            dataIndex: 'dataFlag'
           },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            scopedSlots: { customRender: 'action' },
          }
        ],
		url: {
          list: "/dmmUser/dmmUser/list",
          delete: "/dmmUser/dmmUser/delete",
          deleteBatch: "/dmmUser/dmmUser/deleteBatch",
          exportXlsUrl: "dmmUser/dmmUser/exportXls",
          importExcelUrl: "dmmUser/dmmUser/importExcel",
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