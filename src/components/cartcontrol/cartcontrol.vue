<template>
  <div class="cartcontrol">
  	<div class="cart-decrease" v-show="food.count>0" @click.stop.prevent="decreaseCart" transition="move">
  		<div class="inner icon-remove_circle_outline"></div>
  	</div>
  	<div class="cart-count con-shopping_cart" v-show="food.count>0">{{food.count}}</div>
  	<div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>
<script type='text/ecmascript-6'>
import Vue from 'vue'
export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart (event) {
      if (!event._constructed) {
        return
      }
      if (!this.food.count) {
        Vue.set(this.food, 'count', 1)
      } else {
        this.food.count++
      }
      this.$dispatch('cart.add', event.target)
    },
    decreaseCart (event) {
      if (!event._constructed) {
        return
      }
      if (this.food.count > 0) {
        this.food.count--
      }
    }
  }
}
</script>
<style lang='stylus' rel="stylesheet/stylus">
  .cartcontrol
    font-size: 0
    .cart-decrease
      display: inline-block
      padding: 6px
      transition: all 0.2s linear
      vertical-align: top
      &.move-transition
       opacity: 1
       transform: translate3d(0, 0, 0)
       .inner
          display: inline-block
          line-height: 24px
          font-size: 24px
          transition: all 0.2s linear
          color: rgb(49, 144, 232)
          transform: rotate(0)
      &.move-enter, &.move-leave
        opacity: 0.2
        transform: translate3d(24px, 0, 0)
        .inner
          transform: rotate(180deg)
    .cart-count
      display: inline-block
      font-size: 14px
      line-height: 24px
      color: rgb(147, 153, 159)
      padding-top: 6px
      width: 12px
      text-align: center
    .cart-add
      display: inline-block
      padding: 6px
      line-height: 24px
      font-size: 24px
      vertical-align: top
      color: rgb(49, 144, 232)
</style>
