<template>
  <el-drawer :visible="visible" :size="460" @click="onClose">
    <el-form ref="TriggerForm" :model="TriggerForm" label-width="80px" :rules="rules">
      <el-row>
        <el-form-item label="Function Name" v-if="currentFunction.name" prop="functionName">
          <el-select v-model="TriggerForm.functionName" placeholder="please select a function" style="width: 368px;" size="small">
            <el-option
              v-for="(item,index) in data"
              :key="item.index"
              :label="item.name"
              :value="item.key">
              {{item.name}}
            </el-option>
          </el-select>
        </el-form-item>
      </el-row>
      <el-row>
        <el-form-item label="data" prop="data">
          <el-input
            autosize
            type="textarea"
            placeholder="please enter the data"
            v-model="TriggerForm.data"/>
        </el-form-item>
      </el-row>
    </el-form>
    <el-row type="flex" justify="end">
      <el-button type="primary" :loading="triggering" @click="onSub('TriggerForm')" style="margin-bottom: 24px"> Trigger </el-button>
    </el-row>
    <el-card :extra="triggerResultType" class="box-card">
      <div slot="header" class="clearfix">
        <span>Result</span>
      </div>
      <div :style="{ minHeight: '64px' }" class="text item">
        <span style="word-break: break-all">{{ triggerResult }}</span>
      </div>
    </el-card>
  </el-drawer>
</template>

<script>
  import { triggerFunc } from '@/api/func'

  export default {
    name: 'TriggerVue',
    data () {
      return {
        TriggerForm: {},
        triggerResult: '',
        triggerResultType: '',
        triggering: false,
        rules: {
          functionName: [
            { required: true, message: 'Please select a function!', trigger: 'blur' }
          ]
        }
      }
    },
    props: {
      visible: {
        type: Boolean,
        default: false
      },
      visibleTrigger: {
        type: Boolean,
        default: false
      },
      data: {
        type: Array,
        default: () => []
      },
      currentFunction: {
        type: Object,
        default: () => {}
      }
    },
    methods: {
      onClose () {
        this.$parent.closeTrigger()
      },
      onSub(subName) {
        this.triggering = true
        this.triggerResult = ''
        this.triggerResultType = ''
        this.$refs[subName].validate((err) => {
          if (err) {
            const values = this.TriggerForm
            console.log('Received values of form: ', values, typeof values, values.data, typeof values.data)
            /*triggerFunc(values.functionName, values)
              .then((res) => {
                this.triggerResult = JSON.stringify(res)
                this.triggerResultType = typeof res
              })
              .finally(() => {
                setTimeout(() => {
                  this.triggering = false
                }, 500)
              })*/
            const data = new FormData()
            const functionName = values.functionName
            const functionData = values.data.split(/[\s\n]/)
            console.log(functionData[1], typeof functionData, '33333333333')
            data.append('functionName', JSON.stringify(functionName))
            /*data.append('Data', JSON.stringify(functionData))*/
            for(let i = 0; i<functionData.length;i++){
              console.log(functionData[i])
              data.append( JSON.stringify(i), JSON.stringify(functionData[i]))
            }
            data.forEach((value, key) => {
              console.log("key %s: value %s", key, value);
            })
            const _this = this

            async function onOk() {
              try {
                await triggerFunc(functionName, data)
                  .then((res) => {
                    console.log('res', res, typeof res)
                    _this.triggerResult = JSON.stringify(res.data)
                    _this.triggerResultType = typeof res
                  })
              } catch (error) {
                _this.$message.error('Function trigger failed!')
              } finally {
                setTimeout(() => {
                  _this.triggering = false
                }, 500)
              }
            }
            onOk()
          } else {
            console.log('fail')
          }
        })
      }
    },
    watch: {
      currentFunc () {
        this.TriggerForm.setFieldsValue({
          functionName : this.currentFunction.name
        })
      }
    }
  }
</script>

<style>
  .text {
    font-size: 14px;
  }
  .item {
    padding: 18px 0;
  }
  .clearfix{
    border-bottom: 1px solid;
  }
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }
  .el-form-item, .box-card{
    width: 368px;
    display: block;
    margin-left: 10%;
  }
  /deep/.textarea{
    height: 40px;
  }
  .el-row{
    width: 416px;
  }
</style>
