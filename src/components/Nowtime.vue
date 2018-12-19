<template>
    <div id="time" style="float: right">
      <!--{{nowTime}}-->
    </div>
</template>

<script>
  import $ from 'jquery'
    export default {
        name: "Nowtime",
      mounted(){
        setInterval(function(){
            let year = new Date().getFullYear();
            let month =new Date().getMonth() + 1 < 10? "0" + (new Date().getMonth() + 1): new Date().getMonth() + 1;
            let date =new Date().getDate() < 10? "0" + new Date().getDate(): new Date().getDate();
            let hh =new Date().getHours() < 10? "0" + new Date().getHours(): new Date().getHours();
            let mm =new Date().getMinutes() < 10? "0" + new Date().getMinutes(): new Date().getMinutes();
            let ss =new Date().getSeconds() < 10? "0" + new Date().getSeconds(): new Date().getSeconds();
            $("#time").text(year + "年" + month + "月" + date +"日"+" "+hh+":"+mm+":"+ss);
          }
          ,1000)
      },



    }
</script>

<style scoped>

</style>
