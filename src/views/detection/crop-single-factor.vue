<template>
  <my-content @getInitData="getInitData">
    content
  </my-content>
</template>

<script>
  import MyContent from '@/layout/content'
export default {
    name: "crop-single-factor",
    components: {MyContent},
    methods:{
        getInitData(data) {
            console.log(data);
        }
    }
}
</script>

<style scoped>

</style>
