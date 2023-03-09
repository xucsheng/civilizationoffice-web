<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="点位类型">
              <j-dict-select-tag placeholder="请选择点位类型" v-model="queryParam.pointPositionTypeCode" dictCode="point_position_type"/>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="点位名称">
              <a-input placeholder="请输入点位名称" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="所属区域">
                <j-category-select placeholder="请选择所属区域" v-model="queryParam.areaCode" pcode="B03"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="所属街道">
                <j-category-select placeholder="请选择所属街道" v-model="queryParam.streetCode" pcode="B03"/>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
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
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('点位信息表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
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
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
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

    <sys-point-position-modal ref="modalForm" @ok="modalFormOk"></sys-point-position-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import SysPointPositionModal from './modules/SysPointPositionModal'
  import JDictSelectTag from '@/components/dict/JDictSelectTag.vue'
  import { loadCategoryData } from '@/api/api'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import JCategorySelect from '@comp/jeecg/JCategorySelect'
  import JSuperQuery from '@/components/jeecg/JSuperQuery.vue'

  export default {
    name: 'SysPointPositionList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      JDictSelectTag,
      JCategorySelect,
      SysPointPositionModal,
      JSuperQuery,
    },
    data () {
      return {
        description: '点位信息表管理页面',
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
            title:'点位类型',
            align:"center",
            dataIndex: 'pointPositionTypeCode_dictText'
          },
          {
            title:'点位名称',
            align:"center",
            dataIndex: 'name'
          },
          {
            title:'电信营业厅类型',
            align:"center",
            dataIndex: 'telecomTypeCode_dictText'
          },
          {
            title:'医院等级',
            align:"center",
            dataIndex: 'hospitalLevelCode_dictText'
          },
          {
            title:'宾馆等级',
            align:"center",
            dataIndex: 'hotelLevelCode_dictText'
          },
          {
            title:'所属区域',
            align:"center",
            dataIndex: 'areaCode',
            customRender: (text) => (text ? filterMultiDictText(this.dictOptions['areaCode'], text) : '')
          },
          {
            title:'所属街道',
            align:"center",
            dataIndex: 'streetCode',
            customRender: (text) => (text ? filterMultiDictText(this.dictOptions['streetCode'], text) : '')
          },
          {
            title:'所属社区',
            align:"center",
            dataIndex: 'communityCode',
            customRender: (text) => (text ? filterMultiDictText(this.dictOptions['communityCode'], text) : '')
          },
          {
            title:'文明村等级',
            align:"center",
            dataIndex: 'villageLevelCode_dictText'
          },
          {
            title:'文明社区等级',
            align:"center",
            dataIndex: 'communityGradeCode_dictText'
          },
          {
            title:'社区类别',
            align:"center",
            dataIndex: 'communityCategoryCode_dictText'
          },
          {
            title:'文明乡镇等级',
            align:"center",
            dataIndex: 'townshipLevelCode_dictText'
          },
          {
            title:'乡镇（街道办事处）政府地址）',
            align:"center",
            dataIndex: 'addressMain'
          },
          {
            title:'详细地址',
            align:"center",
            dataIndex: 'address'
          },
          {
            title:'宾馆内饭店/餐厅名称',
            align:"center",
            dataIndex: 'restaurantName'
          },
          {
            title:'主次干道',
            align:"center",
            dataIndex: 'roadTypeCode_dictText'
          },
          {
            title:'居民户数',
            align:"center",
            dataIndex: 'residentCount'
          },
          {
            title:'广场面积（平方米）/文挡墙体面积',
            align:"center",
            dataIndex: 'squareArea'
          },
          {
            title:'联系人',
            align:"center",
            dataIndex: 'contacts'
          },
          {
            title:'联系人电话',
            align:"center",
            dataIndex: 'contactPhone'
          },
          {
            title:'距离（公里）',
            align:"center",
            dataIndex: 'distance'
          },
          {
            title:'周边300米范围是否有居民校区',
            align:"center",
            dataIndex: 'campusFlag_dictText'
          },
          {
            title:'点位图片',
            align:"center",
            dataIndex: 'picture',
            scopedSlots: {customRender: 'imgSlot'}
          },
          {
            title:'经度 ',
            align:"center",
            dataIndex: 'pointLongitude'
          },
          {
            title:'纬度',
            align:"center",
            dataIndex: 'pointLatitude'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/pointposition/sysPointPosition/list",
          delete: "/pointposition/sysPointPosition/delete",
          deleteBatch: "/pointposition/sysPointPosition/deleteBatch",
          exportXlsUrl: "/pointposition/sysPointPosition/exportXls",
          importExcelUrl: "pointposition/sysPointPosition/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
        loadCategoryData({code:'B03'}).then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'areaCode', res.result)
          }
        })
        loadCategoryData({code:'B03'}).then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'streetCode', res.result)
          }
        })
        loadCategoryData({code:'B03'}).then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'communityCode', res.result)
          }
        })
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'pointPositionTypeCode',text:'点位类型',dictCode:'point_position_type'})
        fieldList.push({type:'string',value:'name',text:'点位名称',dictCode:''})
        fieldList.push({type:'string',value:'telecomTypeCode',text:'电信营业厅类型',dictCode:'telecom_type'})
        fieldList.push({type:'string',value:'hospitalLevelCode',text:'医院等级',dictCode:'hospital_level'})
        fieldList.push({type:'string',value:'hotelLevelCode',text:'宾馆等级',dictCode:'hotel_level'})
        fieldList.push({type:'string',value:'areaCode',text:'所属区域'})
        fieldList.push({type:'string',value:'streetCode',text:'所属街道'})
        fieldList.push({type:'string',value:'communityCode',text:'所属社区'})
        fieldList.push({type:'string',value:'villageLevelCode',text:'文明村等级',dictCode:'village_level'})
        fieldList.push({type:'string',value:'communityGradeCode',text:'文明社区等级',dictCode:'community_grade'})
        fieldList.push({type:'string',value:'communityCategoryCode',text:'社区类别',dictCode:'community_category'})
        fieldList.push({type:'string',value:'townshipLevelCode',text:'文明乡镇等级',dictCode:'township_level'})
        fieldList.push({type:'string',value:'addressMain',text:'乡镇（街道办事处）政府地址）',dictCode:''})
        fieldList.push({type:'string',value:'address',text:'详细地址',dictCode:''})
        fieldList.push({type:'string',value:'restaurantName',text:'宾馆内饭店/餐厅名称',dictCode:''})
        fieldList.push({type:'string',value:'roadTypeCode',text:'主次干道',dictCode:'road_type'})
        fieldList.push({type:'string',value:'residentCount',text:'居民户数',dictCode:''})
        fieldList.push({type:'string',value:'squareArea',text:'广场面积（平方米）/文挡墙体面积',dictCode:''})
        fieldList.push({type:'string',value:'contacts',text:'联系人',dictCode:''})
        fieldList.push({type:'string',value:'contactPhone',text:'联系人电话',dictCode:''})
        fieldList.push({type:'string',value:'distance',text:'距离（公里）',dictCode:''})
        fieldList.push({type:'string',value:'campusFlag',text:'周边300米范围是否有居民校区',dictCode:'logic'})
        fieldList.push({type:'string',value:'picture',text:'点位图片',dictCode:''})
        fieldList.push({type:'double',value:'pointLongitude',text:'经度 ',dictCode:''})
        fieldList.push({type:'double',value:'pointLatitude',text:'纬度',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>