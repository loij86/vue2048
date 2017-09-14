<template>
  <div class="modal" rel="modal">

    <div  class="modal-content" >
      <h3 class="c-red">历史最高分数:{{maxScore}}</h3>
      <h1 class="c-orange">
        获得分数: {{gameScore}}
      </h1>
      <div>
        <button class="btn c-red2" @click="restart">重新开始</button>
        <button class="btn c-red1" @click="edit">退出游戏</button>
      </div>
      <div class="share">
        <p>分享到:</p>
        <span>QQ</span> <span>微信</span> <span>微博</span>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    props:['gameScore'],
    data(){
//      var that = this
      return {
        isShowSelf:true

      }
    },
    computed:{
      maxScore(){return localStorage.maxScore}
    },
    methods:{
    /*结束游戏*/
      edit(){
        this.$emit('restartgame');
      },
      /*重新开始*/
      restart(){
//        this.isShowSelf = false;
        this.$emit('restartgame');
      }
    }
  }
</script>
<style scoped>
  .modal{
    position: fixed;
    width:100%;
    height: 100%;
    top: 0;
    left: 0;
    background-color: rgba(1,1,17,0.7);
    z-index: 10;
  }
  .modal-content{
    position: absolute;
    top: 0;
    left: 0;
    right:0;
    bottom: 0;
    margin:auto;
    width:80%;
    max-width:800px;
    height:300px;
    border-radius: 14px;
    z-index:1000;
    background-color:rgba(255,255,255,1);
  }

  .share{
    font-size:16px;
  }
  .share p{
    margin-bottom:4px;
  }
  .share span{
    display: inline-block;
    width:50px;
    height:50px;
    margin-left:10px;
    text-align: center;
    line-height:50px;
    border:1px solid #ccc;
    border-radius: 12px;
    background-color: #e5e5e5;
  }
  .btn{
    width: 100px;
    height:40px;
    line-height:40px;
    border:1px #ccc solid ;
    border-radius: 5px;
    background-color: #fff;
    color:white;
    font-weight:bold;
    font-size:16px;
  }
  /*颜色设置*/
  .c-orange{
    color: orange;
  }
  .c-red{
    color: #ffa474;
  }
  .c-muted{
    color:#ddd;
  }
  .c-red1{
    background-color: #ffaf72;
  }
  .c-red2{
    background-color: #72cfff;
  }
</style>
