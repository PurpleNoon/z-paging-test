<script setup lang="ts">
import NationalityPicker from "@/components/NationalityPicker/index.vue"

const handleSelect = (item: any) => {
  console.log("handleSelect: ", item)
}
</script>

<template>
  <view>
    <view style="height: 100px"></view>
    <NationalityPicker
      @select="handleSelect"
    ></NationalityPicker>
  </view>
</template>

<style lang="scss" scoped></style>
