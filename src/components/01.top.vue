<template>
  <div class="top-container">
    <div class="left-box">
      <div class="icon-wrapper">
        <span class="iconfont icon-home" @click="perhome()"></span>
        <span class="iconfont icon-sami-select"></span>
        <span class="iconfont icon-full-screen"></span>
      </div>
      <div class="history-wrapper">
        <span class="iconfont icon-arrow-lift" @click="perback()"></span>
        <span class="iconfont icon-arrow-right" @click="pergo()"></span>
      </div>
    </div>
    <div class="right-box">
      <div class="el-input el-input--small el-input--prefix">
        <input
          type="text"
          autocomplete="off"
          placeholder="搜索"
          class="el-input__inner"
          v-model="inputValue"
          @keyup.enter="toResult"
        />
        <span class="el-input__prefix">
          <i class="el-input__icon el-icon-search"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'top',
    data() {
      return {
        query: '',
        // 输入内容
        inputValue: ''
      }
    },
    methods:{
      toResult(){
        if(this.inputValue==''){
          this.$message.warning('请输入内容')
        }else{
          this.$router.push('/result?q='+this.inputValue)
        }
      },
      perhome(){
        this.$router.push('/discovery')
      },
      pergo(){
        this.$router.go(1)
      },
      perback(){
        this.$router.go(-1)
      }
    }
  }
</script>

<style scoped></style>
