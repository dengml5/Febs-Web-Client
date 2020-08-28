<template>
    <div>
        <div class="list-item" v-for="(item, index) in list" :key="index">
            <div class="item-name">
                <span class="item-header">key: </span><span>{{item.key}}</span>
                <hr>
                <span class="item-header">groupId: </span><span>{{item.groupId}}</span>
                <hr>
                <span class="item-header">artifactId: </span><span>{{item.artifactId}}</span>
                <hr>
                <span class="item-header">version: </span><span>{{item.version}}</span>
                <hr style="height:1px;border:none;border-top:1.2px solid #000;">
            </div>
            <div v-if="item.children" class="children-item">
                <list :list="item.children"></list>
            </div>
        </div>
    </div>
</template>
<script>
export default {
  name: "List",
  props: {
    list: Array
  }
};
</script>
<style>
.children-item {
    padding: 0 20px
}
.item-name {
    line-height: 20px;
    font-size: 14.08px;
    padding: 0 20px;
}
.item-header {
    color: #67C23A;
}
</style>
