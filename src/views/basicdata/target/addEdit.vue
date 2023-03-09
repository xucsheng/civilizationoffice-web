<template>
  <a-modal :title="title" :width="800" :visible="visible" @cancel="onCancel" @ok="handleSubmit" cancelText="关闭">
    <a-card :bordered="false">
      <a-form-model ref="form" :model="formInline" :rules="rules">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="指标类型" prop="targetType">
              <j-dict-select-tag v-model="formInline.targetType" placeholder="请选择" dictCode="base_target_type" />
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="指标内容" prop="targetName">
              <a-input v-model="formInline.targetName" placeholder="请输入"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="考察方式" prop="inspectType">
              <j-dict-select-tag v-model="formInline.inspectType" placeholder="请选择" dictCode="inspect_Type" />
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row v-show="showInspectUnit">
          <a-col :span="24">
            <a-form-model-item label="单位" prop="inspectUnit">
              <a-input v-model="formInline.inspectUnit" placeholder="处"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="考察标准">
              <a-textarea v-model="formInline.inspectStandard" placeholder="请输入"></a-textarea>
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row
        class="questions-row"
        >
             <span class="label-red">考察问题</span>
          <a-col :span="24"
        v-for="(item, index) in formInline.inspectQuestions.questions" :key="index"
          >
            <a-form-model-item
            :prop="'inspectQuestions.questions[' + index + ']'"
            :rules="[{required: true, message: '请输入', trigger: 'blur'}]"
            >
              <!-- <span class="label-red" slot="label">考察问题</span> -->
              <a-row >
                <a-col :span="20">
                  <a-input placeholder="请输入" 
                
                  v-model="formInline.inspectQuestions.questions[index]"></a-input>
                </a-col>
                <a-col class="add-del-btn" :span="4">
                  <span @click="handleAddQuestion()"><a-icon type="plus-circle" /></span>
                  <span v-if="formInline.inspectQuestions.questions.length !== 1" @click="handleDelQuestion(index)"
                    ><a-icon type="minus-circle"
                  /></span>
                </a-col>
              </a-row>
            </a-form-model-item>
          </a-col>
        </a-row>
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="指标排序" prop="sort">
              <a-input-number :min="minValue" :precision="0" v-model="formInline.sort" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </a-card>
  </a-modal>
</template>

<script>
import { postAction, putAction } from '@/api/manage'
export default {
  props: {
    chooseRow: {
      type: Object,
      default: () => {},
    },
  },
  watch: {
    visible: {
      handler(newVal) {
        if (newVal && this.title == '编辑指标') {
          this.formInline = {
            targetType: this.chooseRow.targetType,
            targetName: this.chooseRow.targetName,
            inspectType: this.chooseRow.inspectType,
            inspectUnit: Number(this.chooseRow.inspectUnit),
            inspectStandard: this.chooseRow.inspectStandard,
            inspectType: this.chooseRow.inspectType,
            inspectQuestions: this.chooseRow.inspectQuestions,
            sort: this.chooseRow.sort,
          }
        } else {
          // 新增指标
          this.formInline = {
            targetType: undefined,
            targetName: '',
            inspectType: undefined,
            inspectStandard: '',
            inspectQuestions: {
              questions: [''],
            },
            sort: '',
          }
          this.$refs.form && this.$refs.form.resetFields()
        }
      },
    },
    'formInline.inspectType': {
      handler(val) {
        this.showInspectUnit = val == '1'
      },
    },
  },
  data() {
    return {
      minValue: 0,
      visible: false,
      title: '',
      showInspectUnit: false,
      formInline: {
        targetType: undefined,
        targetName: '',
        inspectType: undefined,
        inspectUnit: null,
        inspectStandard: '',
        inspectQuestions: {
          questions: [],
        },
      },
      rules: {
        targetType: [{ required: true, message: '请选择!' }],
        targetName: [{ required: true, message: '请输入!' }],
        inspectType: [{ required: true, message: '请选择!' }],
        inspectUnit: [{ required: true, message: '请输入!' }],
        sort: [{ required: true, message: '请输入!' }],
      },
    }
  },
  methods: {
    onCancel() {
      this.visible = false
    },
    handleAddQuestion() {
      this.formInline.inspectQuestions.questions.push('')
    },
    handleDelQuestion(index) {
      this.formInline.inspectQuestions.questions.splice(index, 1)
    },
    handleSubmit(e) {
      e.preventDefault()
      // 过滤处的校验规则
      if (this.formInline.inspectType != '1') {
        delete this.rules.inspectUnit
      } else {
        this.rules.inspectUnit = [{ required: true, message: '请输入!' }]
      }
      this.$refs.form.validate((ok, err) => {
        if (ok) {
          const param = { ...this.formInline }
          if (this.title == '新建指标') {
            delete param.id
            postAction(`/basedata/baseTarget/add`, param).then((res) => {
              if (res.success) {
                this.$message.success(res.message)
                this.visible = false
                this.$parent.$parent.loadDataCopy()
              } else {
                this.$message.warning(res.message)
              }
            })
          } else {
            param.id = this.chooseRow.id
            putAction(`/basedata/baseTarget/edit`, param).then((res) => {
              if (res.success) {
                this.$message.success(res.message)
                this.visible = false
                this.$parent.$parent.loadDataCopy()
              } else {
                this.$message.warning(res.message)
              }
            })
          }
        }
      })
    },
  },
}
</script>
<style scoped lang="less">
@import '~@assets/less/common.less';
/deep/ .ant-modal-body {
  padding: 0px 24px;
}
/deep/ .ant-modal-footer {
  text-align: center;
}
/deep/ .ant-form-item {
  margin-bottom: 10px;
}
.questions-row {
   .ant-form-item {
  margin-bottom: 0px;
}
}
/deep/ .ant-input-number {
  width: 100%;
}
textarea.ant-input {
  min-height: 100px;
}
.label-red::before {
  content: '*';
  display: inline-block;
  color: red;
  margin-right: 4px;
}
.add-del-btn {
  text-align: right;
}
/deep/ .anticon-plus-circle {
  cursor: pointer;
  font-size: 20px;
  margin-right: 20px;
}
/deep/ .anticon-minus-circle {
  cursor: pointer;
  font-size: 20px;
  color: red;
}
</style>

