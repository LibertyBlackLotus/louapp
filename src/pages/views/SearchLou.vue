<script setup>
import loumap from "@/modal/qlhlou.min"
import {reactive,ref} from "vue";
import {get,set,find,forEach,toNumber} from "lodash";

import CMPRelations from "@/components/Relations.vue";

const filter = reactive({
    indexLou:"", // 0005
    username:"王志", // 王志，孟潮，孟祥旺，王凤, 王雷
    house:"", // 47-1-1-101
})

// 是否找到当前用户
const hasFind = ref(false);
// 找到邻居列表，渲染用的
const listNeighbor = ref([]);
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
    listNeighbor.value=[];
    let currentUsers = [];
    // console.log("===search==",filter.indexLou)
    // 选房序号是唯一的可以精确匹配
    if(filter.indexLou){
        // let ret = find(list,{"选房序号":filter.indexLou});
        forEach(listFilterLou,(v)=>{
            let louindex=v["选房序号"],ret = parseInt(louindex)==filter.indexLou?true:false;
            // 找到匹配人就存入
            if(!!ret){
                currentUsers=[v];
            }
        });
    }else if(filter.username){
        // 被腾退人可能多个，需要模糊匹配
        forEach(listFilterLou,(v)=>{
            let ret = v["被腾退人"]==filter.username?true:false;
            // 找到匹配人就存入
            if(!!ret){
                currentUsers.push(v);
            }
        });
    }else if(filter.house){
        forEach(listFilterLou,(v)=>{
            // console.log("==vv=",v);
            // let ret = v["被腾退人"]==filter.username?true:false;
            let ret = find(v.lous,{"unitID":filter.house})
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

</script>
<template>
<div class="page search">
    <div class="filter">
        <div class="field">
            <label>选房序号：</label>
            <a-input allowClear v-model:value="filter.indexLou" placeholder="请输入选房序号" />
        </div>
        <div class="field">
            <label>被腾退人：</label>
            <a-input allowClear v-model:value="filter.username" placeholder="请输入被腾退人姓名" />
        </div>
        <div class="field">
            <label>房间地址：</label>
            <a-input allowClear v-model:value="filter.house" placeholder="如：47-1-1-101" />
        </div>
        <div class="field">
            <a-button type="primary" @click.stop="doSearch">搜索</a-button>
            <p class="tip">(*优先匹配选房序号)</p>
        </div>
    </div>
    <div class="content">
        <a-empty description="未找到相关数据" v-if="!hasFind" />
        <div class="result" v-else>
            <CMPRelations :neighbors="listNeighbor" />
        </div>
    </div>
</div>
</template>
<style lang="less" scoped>
.page.search{
    >.filter{
        display:flex;
        padding:0 10px;
        >.field{
            margin-right:10px;
            display:flex;
            align-items: center;
            >.tip{padding:0;margin:0;margin-left:10px;color:grey;}
            >label{margin-right:5px;min-width:80px;}
            &:last-child{margin-right:0;}
        }
    }
    >.content{
        margin-top:10px;
        display:flex;
        justify-content: center;
    }
}
</style>