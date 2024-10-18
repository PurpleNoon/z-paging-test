<script setup lang="ts">
import { shallowRef, computed, ref } from "vue"
import data from "./data"

defineOptions({
  name: "NationalityPicker",
  options: {
    virtualHost: true,
  },
})

export interface NationalityOption {
  label: string
  value: string
  key: string
}

export interface NationalityOptionWrap {
  index?: string
  data: NationalityOption[]
}

const nationalityOptions =
  shallowRef<NationalityOptionWrap[]>(data)
const allNationalityOptions = computed(() =>
  nationalityOptions.value.flatMap((item) =>
    item.index
      ? [
          {
            key: item.index,
            label: item.index,
            type: "index",
            value: item.index,
          },
          ...item.data,
        ]
      : [...item.data]
  )
)
const nationalityViewVisible = ref(false)

const showPopup = () => {
  nationalityViewVisible.value = true
}

const pagingRef = ref()
const pagingSize = ref(26)
const virtualList = ref<any[]>([])
const virtualListChange = (vList: any[]) => {
  virtualList.value = vList
}

const queryList = async (
  pageNo: number,
  pageSize: number
) => {
  console.log("pageNo:", pageNo, pageSize)
  try {
    pagingRef.value.setLocalPaging(
      allNationalityOptions.value
    )
  } catch (error) {
    console.error(error)
    pagingRef.value.complete(false)
  }
}
const jumpTo = async (
  nationality: NationalityOptionWrap
) => {
  const itemIndex = allNationalityOptions.value.findIndex(
    (item) => item.value === nationality.index
  )
  console.log("itemIndex:", itemIndex)
  const targetPageNo =
    Math.ceil((itemIndex + 1) / pagingSize.value) + 1
  await pagingRef.value?.refreshToPage(targetPageNo)
  await pagingRef.value?.scrollIntoViewByIndex(itemIndex)
}

const emit = defineEmits<{
  (e: "select", value: string): void
}>()
const handleClick = (item: any) => {
  console.log("Get it")
  emit("select", item)
}
</script>

<template>
  <button @click="showPopup">show</button>
  <wd-popup
    v-model="nationalityViewVisible"
    position="bottom"
    safe-area-inset-bottom
  >
    <view class="wrapper" style="position: relative">
      <view
        style="
          position: absolute;
          z-index: 1000;
          top: calc(50% + 25px);
          right: 4px;
          transform: translateY(-50%);
          text-align: center;
        "
      >
        <template
          v-for="nationality in nationalityOptions"
          :key="nationality.index"
        >
          <view
            v-if="nationality.index"
            style="font-size: 12px; padding: 4px 6px"
            @click="jumpTo(nationality)"
          >
            {{ nationality.index }}
          </view>
        </template>
      </view>
      <z-paging
        v-if="nationalityViewVisible"
        ref="pagingRef"
        :fixed="false"
        :default-page-size="pagingSize"
        :force-close-inner-list="true"
        :hide-no-more-inside="true"
        :refresher-enabled="false"
        :show-default-loading-more-text="false"
        :show-loading-more-when-reload="false"
        :show-refresher-when-reload="false"
        cell-height-mode="dynamic"
        lower-threshold="750rpx"
        preload-page="12"
        use-virtual-list
        @query="queryList"
        @virtual-list-change="virtualListChange"
      >
        <view
          style="
            display: flex;
            flex: none;
            justify-content: center;
            align-items: center;
            background-color: #fff;
            border-radius: 16px 16px 0px 0px;
          "
        >
          <text
            style="
              padding: 12px 0 8px 0;
              font-size: 20px;
              line-height: 28px;
              font-weight: bold;
            "
          >
            选择国籍
          </text>
        </view>
        <view
          v-for="item in virtualList"
          :id="`zp-id-${item.zp_index}`"
          :key="item.zp_index"
          style="background-color: #fff"
        >
          <view
            v-if="item.type === 'index'"
            style="
              font-size: 36rpx;
              background-color: #f2f3f5;
              padding: 12px 10px;
            "
          >
            {{ item.label }}
          </view>
          <view
            v-else
            :style="{
              backgroundColor: item.bg || 'transparent',
            }"
            style="font-size: 32rpx; padding: 10px 16px"
            @touchend="item.bg = 'transparent'"
            @touchstart="item.bg = '#f2f2f2'"
            @click="handleClick(item)"
          >
            {{ item.label }}
          </view>
        </view>
      </z-paging>
    </view>
  </wd-popup>
</template>

<style lang="scss" scoped>
.wrapper {
  height: calc(100vh - var(--window-top) - 50px);
  height: calc(
    100vh - var(--window-top) -
      constant(safe-area-inset-bottom) - 50px
  );
  height: calc(
    100vh - var(--window-top) - env(safe-area-inset-bottom) -
      50px
  );
}
</style>
