<script setup>
import loumap from "@/modal/qlhlou.min"
import {reactive,ref} from "vue";
import {get,set,find,forEach,toNumber} from "lodash";

import CMPMobileRelations from "@/components/mobile/Relations.vue";

const filter = ref({
  indexLou:"", // 0005
  username:"", // 王志，孟潮，孟祥旺，王凤, 王雷
  house:"", // 47-1-1-101
})

// 是否找到当前用户
const hasFind = ref(false);
// 找到邻居列表，渲染用的
const listNeighbor = ref([]);
const isShowSearchPro=ref(false);
function showSearchPro(){
  isShowSearchPro.value = true;
}
function hideSearchPro(){
  isShowSearchPro.value = false;
}
/**
 * 把选房信息拆分为房屋数组信息
 * 如那个地块，几号楼，几单元，几室
 */
function parseLouInfo(str){
    // 分号分割楼数
    let _listLou = [];
    const _lous = str.split(";");
    // console.log("===",_lous);
    forEach(_lous,(l)=>{
        if(!l){return;}
        
        // 冒号分割地块，空格分割楼-单元，”单元“分割单元和室
        let lou = {area:"",lou:"",unit:"",room:"",unitID:""},
            _splLou = l.split("："),
            _area = parseInt(_splLou[0]),
            _units = _splLou[1];
        lou.area=_area;
        if(_units){
            const _splitLou = _units.split(" "),
                _house=parseInt(_splitLou[0]),
                _splitUnit=_splitLou[1].split("单元"),
                _roomType = _splitLou[2];

            lou.lou=_house;

            let _unit = toNumber(_splitUnit[0]),
            _room = toNumber(_splitUnit[1]);

            lou.unit = _unit;
            lou.room = _room;
            lou.roomType=_roomType;

            set(lou,`key_${_area}_${_house}_${_unit}_${_room}`)
            set(lou,"unitID",`${_area}-${_house}-${_unit}-${_room}`)
            /**
             * 邻居的unitID
             * UID:up unit id 上楼
             * DID: down unit id 下楼
             * NID: neighbor unit id 邻居
            */  
            set(lou,"unitUID",`${_area}-${_house}-${_unit}-${_room+100}`)
            set(lou,"unitDID",`${_area}-${_house}-${_unit}-${_room-100}`)
            let _nroom=0;
            if(!(_room%2)){  
                _nroom = _room-1;
            }else{
                _nroom = _room+1;
            }
            set(lou,"unitNID",`${_area}-${_house}-${_unit}-${_nroom}`)
        }
        _listLou.push(lou);
    })
    // console.log("===",_listLou);
    return _listLou;
}
// 过滤重复
function filterUniq(){
    let _lous=[],list = loumap.list||[];
    /**
     * 过滤重复的, 优先取选房套数多的
     * 这里重复包括同一个用户多次选择的，还有同名不同户的情况
     * 如果是同一个人需要合并选房信息
     */
    let _usermap = new Map();
    forEach(list,(u)=>{
        let _tu = _usermap.get(u["选房序号"]),
            _cpu=null;
        // 没有重复则加入map
        if(!!_tu){
            const _infolou=_tu["第一轮选房信息"]||"",
                _infolou2=_tu["第2轮选房信息"]||"",
                _infolou3=_tu["第1轮选房信息"]||"";
            // console.log("==tu=",_tu,"===",_infolou,"==",_infolou2,"===",_infolou3)
            if(!_infolou||!_infolou2||!_infolou3){
                if(!_tu["选房信息"]){
                    _tu["选房信息"] = "";
                }
                if(_infolou3){
                    _tu["选房信息"]+=_infolou3;
                    delete _tu["第1轮选房信息"];
                }
                if(_infolou2){
                    _tu["选房信息"]+=_infolou2;
                    delete _tu["第2轮选房信息"];
                }
                if(_infolou){
                    _tu["选房信息"]+=_infolou;
                    delete _tu["第一轮选房信息"];
                }
            }

            _cpu = _tu;

            // console.log("==2==",_tu,"===",u);
        }else{
            if(!u["选房信息"]){
                u["选房信息"] = "";
            }
            const _infolou=u["第一轮选房信息"]||"",
                _infolou2=u["第2轮选房信息"]||"",
                _infolou3=u["第1轮选房信息"]||"";
            if(_infolou3){
                u["选房信息"]+=_infolou3;
                delete u["第1轮选房信息"];
            }
            if(_infolou2){
                u["选房信息"]+=_infolou2;
                delete u["第2轮选房信息"];
            }
            if(_infolou){
                u["选房信息"]+=_infolou;
                delete u["第一轮选房信息"];
            }
            _cpu=u;
            // console.log("==1=====",u);
        }
        // 格式化选房信息
        set(_cpu,"lous",parseLouInfo(_cpu["选房信息"]));
        _usermap.set(u["选房序号"],_cpu)
    })
    // console.log("==map==",list,"===",_usermap);
    _usermap.forEach((u)=>{
        _lous.push(u);
    })
    return _lous;
}

// 过滤好的选房列表
const listFilterLou =filterUniq();

function doFind(unitid){
    let _result = null;
    forEach(listFilterLou,(l)=>{
        let ret = find(l.lous,{unitID:unitid})
        if(ret){
            _result = l;
            return;
        }
    })
    // console.log("==result==",unitid,"==",_result);
    return _result;
}
// 查找邻居
function findNeighbor(users){
    // console.log("===users==",users);
    let _result = [];
    forEach(users,(u)=>{
        forEach(u.lous,(lou)=>{
            let _UNID = get(lou,"unitUID"),
                _DNID = get(lou,"unitDID"),
                _NNID = get(lou,"unitNID"),
                _UID=get(lou,"unitID"),
                _roomType=get(lou,"roomType");
            // 通过unitID找邻居
            // console.log("==v==",_UID,"==",_UNID,"==",_DNID,"==",_NNID);
            const _userUp = doFind(_UNID);
            const _userDown = doFind(_DNID);
            const _userNB = doFind(_NNID);
            _result.push({lou:_UID+" "+_roomType,self:u,up:_userUp,down:_userDown,nb:_userNB});
            // console.log("====",_userUp,"==",_userDown,"==",_userNB)
        })
    })
    return _result;
}
// 找到当前用户
function doSearch(){
  hideSearchPro();
  listNeighbor.value=[];
  let currentUsers = [],_filter = filter.value;
  // console.log("===search==",_filter)
  // 选房序号是唯一的可以精确匹配
  if(_filter.indexLou){
      // let ret = find(list,{"选房序号":_filter.indexLou});
      forEach(listFilterLou,(v)=>{
          let louindex=v["选房序号"],ret = parseInt(louindex)==_filter.indexLou?true:false;
          // 找到匹配人就存入
          if(!!ret){
              currentUsers=[v];
          }
      });
  }else if(_filter.username){
      // 被腾退人可能多个，需要模糊匹配
      forEach(listFilterLou,(v)=>{
          let ret = v["被腾退人"]==_filter.username?true:false;
          // 找到匹配人就存入
          if(!!ret){
              currentUsers.push(v);
          }
      });
  }else if(_filter.house){
      forEach(listFilterLou,(v)=>{
          // console.log("==vv=",v);
          // let ret = v["被腾退人"]==_filter.username?true:false;
          let ret = find(v.lous,{"unitID":_filter.house})
          // console.log("===",ret);
          // // 找到匹配人就存入
          if(!!ret){
              currentUsers.push(v);
          }
      });
  }
  // console.log("==ret 1=",currentUsers)
  // 是否找到
  hasFind.value=(!!currentUsers&&currentUsers.length>0)?true:false;
  // 找到当前用户后再二次匹配周边邻居

  const nbs = findNeighbor(currentUsers);
  // console.log("==nbs===",currentUsers,"===",nbs);

  if(!nbs||nbs.length<=0) return;

  listNeighbor.value = nbs;

  // console.log("====list==",listNeighbor)
}

doSearch();
function doSearchCancel(){
  filter.value={
    indexLou:"",
    username:"",
    house:"",
  }
  hideSearchPro();
}
</script>
<template>
<div class="page_m search">
  <div class="panel search_pro" :class="{'show':isShowSearchPro}">
    <div class="header">
      <h3 class="title">高级检索</h3>
      <button class="btn close" @click.stop="hideSearchPro"></button>
    </div>
    <div class="form">
      <div class="field">
          <label>被腾退人：</label>
          <input v-model="filter.username" placeholder="请输入被腾退人姓名" />
      </div>
      <div class="field">
          <label>房间地址：</label>
          <input v-model="filter.house" placeholder="如：47-1-1-101" />
      </div>
      <div class="field">
        <button class="btn" @click.stop="doSearch">确定</button>
        <button class="btn" @click.stop="doSearchCancel">取消</button>
      </div>
    </div>
  </div>
  <div class="filter">
      <div class="field">
        <label>选房序号：</label>
        <input v-model="filter.indexLou" placeholder="请输入选房序号" />
      </div>
      <div class="field">
        <button class="btn" @click.stop="doSearch">搜索</button>
        <button class="btn" @click.stop="showSearchPro">高级检索</button>
      </div>
  </div>
  <div class="content">
    <a-empty description="未找到相关数据" v-if="!hasFind" />
    <div class="result" v-else>
      <CMPMobile-Relations :neighbors="listNeighbor" />
    </div>
  </div>
</div>
</template>
<style lang="less" scoped>
.page_m.search{
  *{box-sizing: border-box;}
  input,button{
    border:none;
    outline:none;
  }
  button,p{
    padding:0;margin:0;
    background:none;
  }
  button{
    font-size:.14rem;
    position: relative;
    white-space: nowrap;
    text-align: center;
    background-image: none;
    border: 1px solid transparent;
    cursor: pointer;
    -webkit-transition: all .3s cubic-bezier(.645, .045, .355, 1);
    transition: all .3s cubic-bezier(.645, .045, .355, 1);
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -ms-touch-action: manipulation;
    touch-action: manipulation;
    padding: .04rem .15rem;
    border-radius: .04rem;

    color: #fff;
    background-color: #1890ff;
    border-color: #1890ff;
    text-shadow: 0 -1px 0 rgba(0,0,0,0.12);
    -webkit-box-shadow: 0 2px 0 rgba(0,0,0,0.045);
    box-shadow: 0 2px 0 rgba(0,0,0,0.045);
  }
  input{
    outline-style: none ;
    border: 1px solid #ccc; 
    border-radius: .08rem;
    padding: .05rem .1rem;
    font-size: .14rem;
    &:focus{
      border-color: #66afe9;
      outline: 0;
      -webkit-box-shadow: inset 0 1px 1px rgba(0,0,0,.075),0 0 8px rgba(102,175,233,.6);
      box-shadow: inset 0 1px 1px rgba(0,0,0,.075),0 0 8px rgba(102,175,233,.6)
    }
  }
  >.panel.search_pro{
    position:fixed;z-index:999;
    width:60%;height:100%;
    background-color:#eee;
    right:-100%;top:0;
    transition:all .3s linear;
    &.show{
      right:0;
    }
    >.header{
      position:relative;
      border-bottom:1px solid rgba(#aaa,.4);
      >.title{
        text-align:center;
        font-size:.14rem;
        padding:.1rem 0;
        width:100%;
      }
      >.btn.close{
        position:absolute;right:.15rem;top:.1rem;
        font-size:0;padding:0;
        border:none;
        width:.3rem;height:.3rem;
        background:url("data:image/png;base64,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") no-repeat center;
        background-size:cover;
      }
    }
    >.form{
      display:flex;
      flex-direction: column;
      >.field{
        margin-top:.2rem;
        padding:0 .24rem;
        text-align: left;
        >label{
          font-size:.14rem;
          display:block;
        }
        >.btn{
          &:last-child{margin-left:.15rem;}
        }
      }
    }
  }
  >.filter{
    background-color:white;
    position:fixed;
    top:0;left:0;
    z-index:99;
    width:100%;
    height:.8rem;
    display:flex;
    padding:0 .1rem;
    justify-content: center;
    border-bottom:1px solid lightgray;
    >.field{
      margin-right:.1rem;
      display:flex;
      align-items: center;
      >.tip{padding:0;margin:0;margin-left:.1rem;color:grey;}
      >label{
        text-align:center;
        margin-right:.05rem;
      }
      >label,>input{font-size:.14rem;}
      &:last-child{margin-right:0;}
      >.btn{
        font-size:.14rem;
        &:last-child{margin-left:.1rem;}
      }
    }
  }
  >.content{
    padding-top:.8rem;
    display:flex;
    justify-content: center;
    >.result{
      width:100%;
      padding:0 .1rem;
    }
  }
}
</style>