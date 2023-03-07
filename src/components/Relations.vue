<script setup>
import CMPNeighbor from "@/components/Neighbor.vue";
const props = defineProps({
    neighbors:{
        type:Array,
        default:[],
    }
});
</script>
<template>
<div class="cmp relations">
    <a-card :title="nb.lou" class="part" 
        v-for="nb,idx in props.neighbors"
        :key="'nb-'+nb.lou">
        <CMPNeighbor title="楼上" :neighbor="nb.up" />
        <CMPNeighbor title="自己" :neighbor="nb.self" />
        <CMPNeighbor title="楼下" :neighbor="nb.down" />
        <CMPNeighbor title="邻居" :neighbor="nb.nb" />
    </a-card>
</div>
</template>
<style lang="less" scoped>
.cmp.relations{
    display:flex;
    >.part{
        margin-left:10px;
        display:flex;
        flex-direction: column;
        align-items:center;
    }
}
</style>