<template>
  <picker
    mode="multiSelector"
    :value="currentPosition"
    range-key="text"
    :range="positionList"
    @columnchange="handleColumnChange"
  >
    <view class="uni-input">
      {{ positionList[0][currentPosition[0]].text }}，{{ positionList[1][currentPosition[1]] }}，{{
        positionList[2][currentPosition[2]]
      }}
    </view>
  </picker>
</template>
<script setup lang="ts">
import baseList from './list.json'
import {ref,computed} from 'vue'
import _ from 'lodash'

const currentPosition = ref([0, 0, 0])

/* watchEffect(() => {
  try {
    const areaIndex = _.findIndex(props.data, {
      value: props.areaId,
    })
    const xAxisIndex = _.findIndex(props.data[areaIndex].children, {
      value: props.xAxis,
    })
    const yAxisIndex = _.findIndex(props.data[areaIndex].children[xAxisIndex].children, {
      value: props.yAxis,
    })
    /!* console.log([areaIndex, xAxisIndex, yAxisIndex])*!/
    currentPosition.value = [areaIndex, xAxisIndex, yAxisIndex]
  } catch (e) {
    console.error('库位不存在')
    currentPosition.value = [0, 0, 0]
  }
})*/

const transferListToLocationTree = (list) => {
  return _(list)
    .orderBy('areaName')
    .groupBy('areaId')
    .map((xItems, areaId) => {
      return {
        value: areaId,
        text: xItems[0].areaName,
        children: _(xItems)
          .groupBy('xAxis')
          .map((yItems, xAxis) => {
            return {
              value: xAxis,
              text: xAxis,
              children: _.map(yItems, (item) => {
                return {
                  ...item,
                  value: item.yAxis,
                  text: item.yAxis,
                }
              }),
            }
          })
          .value(),
      }
    })
    .value()
}

const list = transferListToLocationTree(baseList)

const positionList = computed(() => {
  if (!currentPosition.value) {
    return []
  }
  const areaList = list.map(({ value, text }) => {
    return {
      value,
      text,
    }
  })
  const xAxisList = list[currentPosition.value[0]].children.map((item) => {
    return item.value
  })
  const yAxisList = list[currentPosition.value[0]].children[currentPosition.value[1]].children.map(
    (item) => {
      return item.value
    }
  )
  console.log('positionList', [areaList, xAxisList, yAxisList])
  return [areaList, xAxisList, yAxisList]
})

const handleColumnChange = (e) => {
  const { column, value } = e.detail
  currentPosition.value[column] = value
  for (let i = column + 1; i < 3; i++) {
    currentPosition.value[i] = 0
  }
}
</script>
