<script setup>
import {ref} from "vue"
import loumap15 from "@/modal/area15"
import loumap18 from "@/modal/area18"
import loumap22 from "@/modal/area22"
import loumap47 from "@/modal/area47"
import loumap48 from "@/modal/area48"
import loumap59 from "@/modal/area59"
import loumap69 from "@/modal/area69"
import loumap63 from "@/modal/area63"
import loumap73 from "@/modal/area73"
import loumap79 from "@/modal/area79"
import _ from "lodash";

const currentUrl = ref("");
function gotoLou(url){
  currentUrl.value=url;
}
const currentMenu = ref(["a15"]);
function doCountLou(area){
  currentMenu.value = ['a'+area];
  gotoLou(`/louapp/dist/#/area${area}`);
}
doCountLou(15)
const updateDataTime=ref(import.meta.env.VITE_UPDATE_DATA_TIME);
</script>
<template>
<a-layout class="area_display">
  <a-layout-header class="header">
    <h3 class="title">各地块选房情况统计</h3>
    <p class="tips">数据更新时间: {{updateDataTime}} by 梦令布孑</p>
  </a-layout-header>
  <a-layout-content class="container">
    <a-menu class="menus" :selectedKeys="currentMenu">
      <a-menu-item key="a15" @click.stop="doCountLou(15)">#15地块</a-menu-item>
      <a-menu-item key="a18" @click.stop="doCountLou(18)">#18地块</a-menu-item>
      <a-menu-item key="a22" @click.stop="doCountLou(22)">#22地块</a-menu-item>
      <a-menu-item key="a47" @click.stop="doCountLou(47)">#47地块</a-menu-item>
      <a-menu-item key="a48" @click.stop="doCountLou(48)">#48地块</a-menu-item>

      <a-menu-item key="a59" @click.stop="doCountLou(59)">#59地块</a-menu-item>
      <a-menu-item key="a63" @click.stop="doCountLou(63)">#63地块</a-menu-item>
      <a-menu-item key="a69" @click.stop="doCountLou(69)">#69地块</a-menu-item>
      <a-menu-item key="a73" @click.stop="doCountLou(73)">#73地块</a-menu-item>
      <a-menu-item key="a79" @click.stop="doCountLou(79)">#79地块</a-menu-item>
    </a-menu>
    <div class="content">
      <iframe class="ifr" :src="currentUrl"></iframe>
    </div>
  </a-layout-content>
</a-layout>
</template>
<style lang="less">
.ant-layout-header{
  height:40px;line-height:40px;
}
</style>
<style lang="less" scoped>
.area_display{
  position:fixed;top:0;left:0;
  width:100%;height:100%;
  .header{
    position:relative;
    >.title{
      color:whitesmoke;
    }
    >.tips{
      color:white;
      position:absolute;right:10px;top:50%;
      transform: translateY(-50%);
      font-size:12px;
    }
  }
  .container{
    >.menus{
      float:left;
      margin-right:10px;
    }
    >.content{
      overflow:hidden;
      max-height:100%;height:100%;
      overflow-y:auto;
      >.ifr{
        border:none;
        width:100%;height:100%;
      }
    }
  }
}
</style>