<template>
  <div>
    <x-form ref="xForm" v-model="formData" :config="formConfig">
      <template #executionListener>
        <el-badge :value="executionListenerLength">
          <el-button size="small" @click="dialogName = 'executionListenerDialog'">编辑</el-button>
        </el-badge>
      </template>
      <template #conditionExpressionBtn>
        <el-button size="small" @click="generateExpression">生成</el-button>
        <el-button size="small" @click="clearExpression">清空</el-button>
      </template>
    </x-form>
    <executionListenerDialog
      v-if="dialogName === 'executionListenerDialog'"
      :element="element"
      :modeler="modeler"
      @close="finishExecutionListener"
    />
  </div>
</template>

<script>
import mixinPanel from '../../common/mixinPanel'
import mixinExecutionListener from '../../common/mixinExecutionListener'
import { commonParse, conditionExpressionParse } from '../../common/parseElement'
export default {
  mixins: [mixinPanel, mixinExecutionListener],
  data() {
    return {
      formData: {}
    }
  },
  computed: {
    formConfig() {
      return {
        inline: false,
        item: [
          {
            xType: 'input',
            name: 'id',
            label: '节点编码',
            rules: [{ required: true, message: '节点编码不能为空' }]
          },
          {
            xType: 'input',
            name: 'name',
            label: '节点名称',
            rules: [{ required: true, message: '节点名称 不能为空' }]
          },
          {
            xType: 'input',
            name: 'documentation',
            label: '节点描述'
          },
          {
            xType: 'slot',
            name: 'executionListener',
            label: '执行监听器'
          },
          {
            xType: 'input',
            name: 'conditionExpression',
            label: '跳转条件',
            disabled: true
          },
          {
            xType: 'slot',
            name: 'conditionExpressionBtn',
            label: '跳转条件生成'
          },
          {
            xType: 'input',
            name: 'skipExpression',
            label: '跳过表达式'
          }
        ]
      }
    }
  },
  watch: {
    'formData.conditionExpression': function(val) {
      if (val) {
        const newCondition = this.modeler.get('moddle').create('bpmn:FormalExpression', { body: val })
        this.updateProperties({ conditionExpression: newCondition })
      } else {
        this.updateProperties({ conditionExpression: undefined })
      }
    },
    'formData.skipExpression': function(val) {
      if (val === '') val = null
      this.updateProperties({ 'flowable:skipExpression': val })
    }
  },
  created() {
    let cache = commonParse(this.element)
    cache = conditionExpressionParse(cache)
    this.formData = cache
  },
  methods: {
    // 生成表达式
    generateExpression(){
      this.formData.conditionExpression = "${conditionVerify.verifyExpression(execution,'"+this.formData.id+"')}"
      const newCondition = this.modeler.get('moddle').create('bpmn:FormalExpression', { body: "${conditionVerify.verifyExpression(execution,'"+this.formData.id+"')}" })
      this.updateProperties({ conditionExpression:  newCondition})
      let cache = commonParse(this.element)
      cache = conditionExpressionParse(cache)
      this.formData = cache
    },
    // 清空表达式
    clearExpression(){
      this.formData.conditionExpression = undefined
    }
  }
}
</script>

<style></style>
