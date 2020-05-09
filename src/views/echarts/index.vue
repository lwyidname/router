<template >
  <div>
   <mapCharts/>
  </div>
</template>
<script>
import mapCharts from './modules/map'
export default {
  name:'EchartsModel',
  components:{mapCharts},
  props:[],
  data(){
    return {
    }
  },
  watch:{},
  computed:{},
  mounted(){},
  methods:{}
}
</script>
<style lang="sass" scoped>

</style>