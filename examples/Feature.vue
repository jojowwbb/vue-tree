<template>
  <div class="container">
    <!-- 基本用法 -->
    <div class="panel">
      <div class="header">基本用法</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree
              ref="basicTreeRef"
              :data="basicUsage"
              :nodeClassName="node => `generated-class-${node.id}`"
            />
          </div>
        </div>
        <div class="desc">
          <div>纯展示</div>
          <div class="desc-block">
            <button @click="handleExpandAll">展开全部节点</button>
            <button @click="handleCollapseAll">折叠全部节点</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 数据正确性验证 -->
    <!-- <div class="panel">
      <div class="header">数据正确性验证</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree :data="orderData" default-expand-all></VTree>
          </div>
        </div>
        <div class="desc">数据正确性</div>
      </div>
    </div> -->

    <!-- 单选 -->
    <div class="panel">
      <div class="header">单选</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree
              v-model="selectableValue"
              :data="selectable"
              @update:modelValue="() => {}"
              selectable
            ></VTree>
          </div>
        </div>
        <div class="desc">
          单选模式。设置 selectable 即可<br />
          v-model: <br />
          {{ selectableValue }}
        </div>
      </div>
    </div>

    <!-- 多选 -->
    <div class="panel">
      <div class="header">多选</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree
              v-if="showCheckable"
              v-model="checkableValue"
              :data="checkable"
              checkable
              :ignore-mode="checkableIgnoreMode"
              :cascade="checkableCascade"
            ></VTree>
          </div>
        </div>
        <div class="desc">
          <div class="desc-block">
            <p>多选模式。设置 checkable 即可</p>
            v-model: <br />
            {{ checkableValue }}
          </div>
          <div class="desc-block">
            <p>
              设置 ignoreMode 可指定 v-model 与 getCheckedNodes
              方法要忽略父节点或者子节点，该 prop 仅初始设置有效
            </p>
            <button
              v-for="mode in (['none', 'parents', 'children'] as IgnoreType[])"
              :key="mode"
              @click="onIgnoreBtnClick(mode)"
            >
              {{ mode }}
            </button>
            <p>当前 ignoreMode: {{ checkableIgnoreMode }}</p>
          </div>
          <div class="desc-block">
            <p>设置 cascade 可指定父子节点是否级联</p>
            <button
              v-for="(mode, index) in [true, false]"
              :key="index"
              @click="onCascadeBtnClick(mode)"
            >
              {{ mode }}
            </button>
            <p>当前 cascade: {{ checkableCascade }}</p>
          </div>
          <div class="desc-block">
            <p>重置以上选项</p>
            <button @click="onResetBtnClick">Reset</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 单选与多选并存 -->
    <div class="panel">
      <div class="header">单选与多选并存</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree
              v-model="bothValue"
              :data="both"
              checkable
              selectable
            ></VTree>
          </div>
        </div>
        <div class="desc">
          当既可以单选又可以多选时， v-model 绑定的是多选的值<br />
          v-model: <br />
          {{ bothValue }}
        </div>
      </div>
    </div>

    <!-- 远程 -->
    <div class="panel">
      <div class="header">远程</div>
      <div class="body">
        <div class="interface">
          <div style="height: 300px">
            <VTree v-if="remoteShow" :load="remoteLoad"></VTree>
          </div>
        </div>
        <div class="desc">
          <div class="desc-block">
            设置 load 方法可以使用远程加载数据，如果有设置 data ，则 data
            数据作为根数据；<br />
            如果没有传 data ，则初始化时调用 load 方法载入根数据，其中节点参数为
            null
          </div>
          <div class="desc-block">
            <button @click="onLoadDataClick">加载树组件</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import VTree, { TreeNode } from '../src'
import { IgnoreType } from '../src/types'
import treeDataGenerator from '../tests/tree-data-generator'
import { defineComponent, ref, nextTick } from 'vue-demi'

const genData = (extra = {}) => {
  return treeDataGenerator(
    Object.assign(
      {
        treeDepth: 3,
        nodesPerLevel: 5,
        sameIdTitle: true
      },
      extra
    )
  )
}

const genChildrenData = (nodeCount = 2) => {
  return treeDataGenerator({
    treeDepth: 1,
    nodesPerLevel: nodeCount,
    inOrder: true,
    useNanoID: true,
  })
}

export default defineComponent({
  name: 'Feature',
  components: {
    VTree
  },
  setup() {
    const selectableData = genData().data
    selectableData[0].selected = true
    const checkableData = genData().data
    checkableData[0].expand = true
    checkableData[1].children![0].disabled = true
    // checkableData[1].children[0].children[0].checked = true
    const selectableValue = ref('')
    const checkableValue = ref<(string | number)[]>([checkableData[0].id!])
    const basicUsage = ref(genData().data)
    const orderData = ref(genData({ inOrder: true }).data)
    const selectable = ref(selectableData)
    const showCheckable = ref(true)
    const checkable = ref(checkableData)
    const checkableIgnoreMode = ref<IgnoreType>('none')
    const checkableCascade = ref(true)
    const both = ref(genData().data)
    const bothValue = ref([])
    const remoteShow = ref(false)
    const remoteLoad = (
      node: TreeNode | null,
      resolve: Function,
      reject: Function
    ) => {
      setTimeout(() => {
        resolve(genChildrenData(node ? 2 : 5).data)
      }, 1000)
    }

    function onIgnoreBtnClick(mode: IgnoreType) {
      checkableIgnoreMode.value = mode
      showCheckable.value = false
      nextTick(() => {
        checkableValue.value = []
        showCheckable.value = true
      })
    }

    function onCascadeBtnClick(mode: boolean) {
      checkableCascade.value = mode
      showCheckable.value = false
      nextTick(() => {
        checkableValue.value = []
        showCheckable.value = true
      })
    }

    function onResetBtnClick() {
      showCheckable.value = false
      nextTick(() => {
        checkableIgnoreMode.value = 'none'
        checkableCascade.value = true
        checkableValue.value = []
        showCheckable.value = true
      })
    }

    function onLoadDataClick() {
      remoteShow.value = false
      nextTick(() => {
        remoteShow.value = true
      })
    }

    const basicTreeRef = ref()

    const handleExpandAll = () => {
      basicTreeRef.value.setExpandAll(true)
    }

    const handleCollapseAll = () => {
      basicTreeRef.value.setExpandAll(false)
      basicTreeRef.value.scrollTo(basicUsage.value[0].id, 0)
    }

    return {
      // 基本用法
      basicUsage,
      basicTreeRef,
      handleExpandAll,
      handleCollapseAll,

      // 数据正确性
      orderData,

      // 单选
      selectable,
      // selectableValue: selectableData[0].id,
      selectableValue,

      // 多选
      showCheckable,
      checkable,
      checkableValue,
      checkableIgnoreMode,
      checkableCascade,

      // 单选与多选并存
      both,
      bothValue,

      // 远程
      remoteShow,
      remoteLoad,
      onIgnoreBtnClick,
      onCascadeBtnClick,
      onResetBtnClick,
      onLoadDataClick
    }
  }
})
</script>

<style lang="less" scoped>
.container {
  width: 100%;
  height: 100%;
  padding: 10px;
  box-sizing: border-box;

  .panel {
    width: 100%;
    margin-bottom: 10px;
    border: 1px solid lightgray;
    border-radius: 5px;

    .header {
      height: 30px;
      border-bottom: 1px solid lightgray;
      padding: 10px 30px;
    }

    .body {
      display: flex;

      .interface {
        flex: 1;
        padding: 10px 30px;
        border-right: 1px solid lightgray;
      }

      .desc {
        flex: 1;
        padding: 10px 30px;

        .desc-block {
          padding: 5px 0;
          margin-bottom: 10px;
          border-bottom: 1px solid lightgray;

          &:last-child {
            border-bottom: none;
          }
        }
      }
    }
  }
}
</style>
