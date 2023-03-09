<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="点位类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['pointPositionTypeCode', validatorRules.pointPositionTypeCode]" :trigger-change="true" dictCode="point_position_type" placeholder="请选择点位类型" @change="ChangePointPositionType" />
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode!='003'">
            <a-form-item label="点位名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['name']" placeholder="请输入点位名称"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='026' ">
            <a-form-item label="电信营业厅类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['telecomTypeCode']" :trigger-change="true" dictCode="telecom_type" placeholder="请选择电信营业厅类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='024' ">
            <a-form-item label="医院等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['hospitalLevelCode']" :trigger-change="true" dictCode="hospital_level" placeholder="请选择医院等级" />
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='020' ">
            <a-form-item label="宾馆等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['hotelLevelCode']" :trigger-change="true" dictCode="hotel_level" placeholder="请选择宾馆等级" />
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode !='005' || pointPositionTypeCode !='006'">
            <a-form-item label="所属区域" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-category-select v-decorator="['areaCode']" pcode="B03" placeholder="请选择所属区域"  />
            </a-form-item>
          </a-col>
          <a-col :span="24"
                 v-show="
                 pointPositionTypeCode =='002'
                 ||pointPositionTypeCode =='003'
                 || pointPositionTypeCode =='004'
                 "

          >
            <a-form-item label="所属街道" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-category-select v-decorator="['streetCode']" pcode="B03" placeholder="请选择所属街道"  />
            </a-form-item>
          </a-col>
          <a-col :span="24"
                 v-show="pointPositionTypeCode =='004'"
          >
            <a-form-item label="所属社区" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-category-select v-decorator="['communityCode']" pcode="B03" placeholder="请选择所属社区"  />
            </a-form-item>
          </a-col>
          <a-col :span="24"
                 v-show="pointPositionTypeCode =='003'"

          >
            <a-form-item label="社区类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['communityCategoryCode']" :trigger-change="true" dictCode="community_category" placeholder="请选择社区类别" />
            </a-form-item>
          </a-col>

          <a-col :span="24"
                 v-show="pointPositionTypeCode =='018'"
          >
            <a-form-item label="文明村等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['villageLevelCode']" :trigger-change="true" dictCode="village_Level" placeholder="请选择文明村等级" />
            </a-form-item>
          </a-col>

          <a-col :span="24"
                 v-show="pointPositionTypeCode =='003'"

          >
            <a-form-item label="文明社区等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['communityGradeCode']" :trigger-change="true" dictCode="community_grade" placeholder="请选择文明社区等级" />
            </a-form-item>
          </a-col>

          <a-col :span="24" v-show="pointPositionTypeCode =='017'">
            <a-form-item label="文明乡镇等级" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['townshipLevelCode']" :trigger-change="true" dictCode="township_level" placeholder="请选择文明乡镇等级" />
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='017' || pointPositionTypeCode =='018'">
            <a-form-item label="乡镇（街道办事处）政府地址）" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['addressMain']" placeholder="请输入乡镇（街道办事处）政府地址）"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24"
                 v-show="pointPositionTypeCode =='005'"
          >
            <a-form-item label="主次干道" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['roadTypeCode']" :trigger-change="true" dictCode="road_type" placeholder="请选择主次干道" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="详细地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['address', validatorRules.address]" placeholder="请输入详细地址"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='020' ">
            <a-form-item label="宾馆内饭店/餐厅名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['restaurantName']" placeholder="请输入宾馆内饭店/餐厅名称"  ></a-input>
            </a-form-item>
          </a-col>

          <a-col :span="24"
                 v-show="pointPositionTypeCode =='003'"
          >
            <a-form-item label="居民户数" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['residentCount', validatorRules.residentCount]" placeholder="请输入居民户数"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24"
                 v-show="pointPositionTypeCode =='007'  || pointPositionTypeCode =='008' ||pointPositionTypeCode =='028' "
          >
            <a-form-item label="广场面积（平方米）/文挡墙体面积" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['squareArea', validatorRules.squareArea]" placeholder="请输入广场面积（平方米）/文挡墙体面积"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode == '001'||pointPositionTypeCode =='007'  || pointPositionTypeCode =='008'">
            <a-form-item label="联系人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['contacts']" placeholder="请输入联系人"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode == '001'||pointPositionTypeCode =='007'  || pointPositionTypeCode =='008'">
            <a-form-item label="联系人电话" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['contactPhone', validatorRules.contactPhone]" placeholder="请输入联系人电话"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='014'|| pointPositionTypeCode =='016' ||pointPositionTypeCode =='017' ||pointPositionTypeCode =='018'" >
            <a-form-item label="距离（公里）" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['distance', validatorRules.distance]" placeholder="请输入距离（公里）"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24" v-show="pointPositionTypeCode =='015'">
            <a-form-item label="周边300米范围是否有居民校区" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['campusFlag']" :trigger-change="true" dictCode="logic" placeholder="请选择周边300米范围是否有居民校区" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="点位图片" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-image-upload isMultiple  v-decorator="['picture']" ></j-image-upload>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="经度 " :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['pointLongitude']" placeholder="请输入经度 " style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="纬度" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['pointLatitude']" placeholder="请输入纬度" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JFormContainer from '@/components/jeecg/JFormContainer'
  import JImageUpload from '@/components/jeecg/JImageUpload'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  import JCategorySelect from '@/components/jeecg/JCategorySelect'

  export default {
    name: 'SysPointPositionForm',
    components: {
      JFormContainer,
      JImageUpload,
      JDictSelectTag,
      JCategorySelect,
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
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
        validatorRules: {
          pointPositionTypeCode: {
            rules: [
              { required: true, message: '请输入点位类型!'},
            ]
          },
          address: {
            rules: [
              { required: true, message: '请输入详细地址!'},
            ]
          },
          residentCount: {
            rules: [
              { required: false},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
          squareArea: {
            rules: [
              { required: false},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
          contactPhone: {
            rules: [
              { required: false},
              { pattern: /^1[3456789]\d{9}$/, message: '请输入正确的手机号码!'},
            ]
          },
          distance: {
            rules: [
              { required: false},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
        },
        url: {
          add: "/pointposition/sysPointPosition/add",
          edit: "/pointposition/sysPointPosition/edit",
          queryById: "/pointposition/sysPointPosition/queryById"
        },
        pointPositionTypeCode:'',
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
    },
    methods: {
      ChangePointPositionType(val){
      this.pointPositionTypeCode = val;
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        debugger;
        this.pointPositionTypeCode = this.model.pointPositionTypeCode;
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'pointPositionTypeCode','name','telecomTypeCode','hospitalLevelCode','hotelLevelCode','areaCode','streetCode','communityCode','communityGradeCode','villageLevelCode','communityCategoryCode','townshipLevelCode','addressMain','address','restaurantName','roadTypeCode','residentCount','squareArea','contacts','contactPhone','distance','campusFlag','picture','pointLongitude','pointLatitude'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
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
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }

        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'pointPositionTypeCode','name','telecomTypeCode','hospitalLevelCode','hotelLevelCode','areaCode','streetCode','communityCode','communityGradeCode','communityCategoryCode','townshipLevelCode','addressMain','address','restaurantName','roadTypeCode','residentCount','squareArea','contacts','contactPhone','distance','campusFlag','picture','pointLongitude','pointLatitude'))
      },
      handleCategoryChange(value,backObj){
        this.form.setFieldsValue(backObj)
      }
    }
  }
</script>