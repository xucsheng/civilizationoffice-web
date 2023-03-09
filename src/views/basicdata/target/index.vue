<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQueryCopy">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="指标类型">
              <j-dict-select-tag v-model="formInline.targetType" placeholder="请选择" dictCode="base_target_type" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="指标内容">
              <a-input 
              
              
               placeholder="请输入" v-model="formInline.targetName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="指标状态">
              <a-select v-model="formInline.useState" placeholder="请选择" allowClear>
                <a-select-option v-for="d in useStateList" :key="d.valud" :value="d.value">{{
                  d.label
                }}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <span style="float: left; overflow: hidden" class="table-page-search-submitButtons">
            <a-col :md="6" :sm="24">
              <a-button type="primary" @click="searchQueryCopy">查询</a-button>
              <a-button style="margin-left: 8px" @click="searchResetCopy">重置</a-button>
            </a-col>
          </span>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新建指标</a-button>
      <a-button v-if="selectedRowKeys.length > 0" style="margin-left: 8px" @click="batchDel">
        批量删除 <a-icon type="down"
      /></a-button>
    </div>

    <!--table区 -->
    <div>
      <!--已选择的清空 -->
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px">
        <i class="anticon anticon-info-circle ant-alert-icon"></i>已选择&nbsp;<a style="font-weight: 600">{{
          selectedRowKeys.length
        }}</a
        >项&nbsp;&nbsp;
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
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <span slot="rowIndex" slot-scope="text, record, index">
          <span>{{ (ipagination.current - 1) * ipagination.pageSize + index + 1 }}</span>
        </span>
        <!-- 字符串超长截取省略号显示-->
        <span slot="esContent" slot-scope="text">
          <j-ellipsis :value="text" :length="20" />
        </span>
        <span slot="inspectStandard" slot-scope="text, record">
          <j-ellipsis :value="text" :length="60" />
        </span>
        <span slot="useState" slot-scope="text, record" :class="`useState_${record.useState}`">
          {{ record.useState == 0 ? '启用' : '禁用' }}
        </span>
        <span slot="action" slot-scope="text, record">
          <a @click="handleEditCopy(record)">编辑</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
          <a-divider type="vertical" />
          <a-popconfirm v-if="record.useState == -1" title="确定启用吗?" @confirm="() => changeUseState(record)">
            <a class="btn-open">启用</a>
          </a-popconfirm>
          <a-popconfirm v-if="record.useState == 0" title="确定禁用吗?" @confirm="() => changeUseState(record)">
            <a class="btn-close">禁用</a>
          </a-popconfirm>
        </span>
      </a-table>
    </div>
    <!-- 新增/编辑 -->
    <AddEdit ref="addEditRef" :chooseRow="chooseRow" />
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { putAction } from '@/api/manage'
import AddEdit from './addEdit.vue'
export default {
  name: 'DataLogList',
  mixins: [JeecgListMixin],
  components: {
    AddEdit,
  },
  data() {
    return {
      description: '指标管理页面',
      disableMixinCreated: true,
      formInline: {
        targetType: undefined,
        targetName: '',
        useState: undefined,
      },
      isorter: {
        // column: 'type,sort,id',
        // order: 'asc',
      },
      useStateList: [
        { label: '启用', value: 0 },
        { label: '禁用', value: -1 },
      ],
      chooseRow: {},
      //表头
      columns: [
        {
          title: '序号',
          dataIndex: '',
          key: 'rowIndex',
          width: 80,
          align: 'center',
          scopedSlots: { customRender: 'rowIndex' },
          // customRender: function (t, r, index) {
          //   return parseInt(index) + 1
          // },
        },
        {
          title: '指标类型',
          align: 'center',
          width: 100,
          dataIndex: 'targetType',
        },
        {
          title: '指标内容',
          align: 'center',
          with: 300,
          dataIndex: 'targetName',
        },
        {
          title: '考核标准',
          align: 'center',
          dataIndex: 'inspectStandard',
           scopedSlots: { customRender: 'inspectStandard' },
        },
        {
          title: '指标状态',
          align: 'center',
          width: 100,
          dataIndex: 'useState',
          scopedSlots: { customRender: 'useState' },
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          width: 200,
          // fixed: 'right',
          scopedSlots: { customRender: 'action' },
        },
      ],
      url: {
        list: '/basedata/baseTarget/list',
        delete: '/basedata/baseTarget/delete',
        deleteBatch: '/basedata/baseTarget/deleteBatch',
      },
    }
  },
  mounted() {
    this.loadDataCopy()
  },
  methods: {
    loadDataCopy() {
      this.queryParam = {
        targetType: this.formInline.targetType,
        targetName: (this.formInline.targetName && `*${this.formInline.targetName}*`) || '',
        useState: this.formInline.useState,
      }
      this.loadData()
    },
    searchQueryCopy() {
      this.loadDataCopy()
    },
    searchResetCopy() {
      this.formInline = {}
      this.loadDataCopy()
    },
    handleAdd() {
      this.$refs.addEditRef.title = '新建指标'
      this.$refs.addEditRef.visible = true
    },
    changeUseState(row) {
      // useState 0: 启用 -1：禁用
      const { id, useState } = row
      const useState2 = useState == 0 ? -1 : 0
      putAction(`/basedata/baseTarget/useState/${id}/${useState2}`)
        .then((res) => {
          if (res.success) {
            this.$message.success(res.message)
            this.loadDataCopy()
          } else {
            this.$message.error(res.message)
          }
        })
        .finally(() => {})
    },
    handleClose(record) {},
    handleEditCopy(row) {
      this.chooseRow = row
      this.$refs.addEditRef.title = '编辑指标'
      this.$refs.addEditRef.visible = true
    },
  },
}
</script>
<style scoped>
@import '~@assets/less/common.less';
.btn-open {
  color: #81b337;
}
.btn-close {
  color: #c15b4e;
}
.useState_0::before {
  content: '';
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #81b337;
}
.useState_-1::before {
  content: '';
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #c15b4e;
}
</style>