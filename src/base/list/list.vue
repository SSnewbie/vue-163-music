<template>
  <div class="list">
    <div class="title-bar">
      <h2 class="title">{{data.title}} <i class="fa fa-angle-right fa-lg" aria-hidden="true"></i></h2>
    </div>
    <div class="content">
      <ul class="item-list">
        <li @click="selectItem(item)" class="item" :class="itemClass + item.size" v-for="item in data.list">
          <img class="img" v-lazy="item.picUrl" width="100" height="120">
          <p class="text">{{item.name}}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    props: {
      data: {
        type: Object,
        default: function() {
          return {
            title: '',
            list: []
          };
        }
      }
    },
    created() {
      this.itemClass = 'item-';
    },
    methods: {
      selectItem(item) {
        this.$emit('select', item);
      }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"
  .list
    position relative
    width 100%
    .title-bar
      padding 10px 0
      .title
        padding-left 8px
        font-weight 500
        font-size $font-size-medium
        border-left 2px solid $color-active
    .content
      position relative
      .item-list
        display flex
        flex-direction row
        flex-wrap wrap
        .item
          margin 5px 0
          &.item-1
            width 100%
          &.item-2
            width 50%
          &.item-3
            width 33.33%
          .img
            width 100%
          .text
            margin 5px
            font-size $font-size-small
            text-ellipsis()
</style>
