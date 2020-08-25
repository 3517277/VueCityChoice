<template>
  <div>
    {{city}}
    <a-divider orientation="left">
      出差概况
    </a-divider>
    <select-city class="city" @cityChanged="change" ref="cityChild" placeholder="请选择省市"></select-city>
    <select-city class="city" @cityChanged="change" multiple ref="cityChild2" placeholder="请选择省市"></select-city>
    
  </div>
  
</template>

<script>
import SelectCity from '@/components/SelectedCity'

export default {
  data(){
    return {
      city:[]
    }
  },
  components:{
    SelectCity
  },
  methods:{
    getCityArr(data) {
    this.selectedCityLists = data
    },
    change(e){
      this.city = e
      console.log(e)
    }
  }
}
</script>

<style lang="less">
.city{
  width: 600px;
}
</style>
