<template>
<div>
    <el-calendar v-model="value">
    </el-calendar>
</div>
</template>

<script>
export default {
  data () {
    return {
      value: new Date()
    }
  },
  methods: {}
}
</script>

<style scoped lang="less">

</style>
