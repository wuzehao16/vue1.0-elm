<template>
	<div class="shopcart">
		<div class="content">
			<div class="content-left" @click="toggleList">
				<div class="logo-wrapper">
					<div class="logo" :class="{highlight:totalCount>0}">
						<span class="icon-shopping_cart" :class="{highlight:totalCount>0}"></span>
					</div>
					<div class="num" v-show="totalCount>0">{{totalCount}}</div>
				</div>
				<div class="price" :class="{highlight:totalPrice>0}">
					￥{{totalPrice}}
				</div>
				<div class="desc" v-show="totalPrice>0">
					配送费￥{{deliveryPrice}}
				</div>
			</div>
			<div class="content-right">
				<div class="pay" :class="{enough:totalPrice>=this.minPrice}" @click='pay'>
					{{payDesc}}
				</div>
			</div>
			<div class="ball-container">
				<div v-for="ball in balls" transition="drop" v-show="ball.show" class="ball">
					<div class="inner inner-hook"></div>
				</div >
			</div>
			<div class="shopcart-list" v-show="listShow" transition="fold">
				<div class="list-header">
					<h1 class="name">购物车</h1>
					<div class="empty" @click="empty">清空</div>
				</div>
				<div class="list-content" v-el:list-content>
					<ul>
						<li v-for="food in selectFoods" class="food">
							<span class="name">{{food.name}}</span>
							<span class="price">￥{{food.price*food.count}}</span>
							<div class="cartcontrol-wrapper">
								<cartcontrol :food="food"></cartcontrol>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</div>
	</div>
	<div class="list-mask" transition="fade" v-show="listShow" @click="hideList"></div>
</template>
<script type='text/ecmascript-6'>
import cartcontrol from '../cartcontrol/cartcontrol'
import BScroll from 'better-scroll'
export default {
  props: {
    selectFoods: {
      type: Array,
      default () {
        return []
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: [],
      fold: true
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return '￥' + this.minPrice + '元起送'
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        console.log(diff)
        return '还差￥' + diff + '元起送'
      } else {
        return '去结算'
      }
    },
    listShow () {
      console.log(1)
      if (!this.totalCount) {
        this.fold = true
        return false
      }
      let show = !this.fold
      if (show) {
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$els.listContent, {
              click: true
            })
          } else {
            this.scroll.refresh()
          }
        })
      }
      return show
    }
  },
  methods: {
    drop (el) {
      console.log(el)
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBalls.push(ball)
          return
        }
      }
    },
    toggleList () {
      if (!this.totalCount) {
        return
      }
      this.fold = !this.fold
    },
    empty () {
      this.selectFoods.forEach((food) => {
        food.count = 0
      })
    },
    hideList () {
      this.fold = true
    },
    pay () {
      if (this.totalPrice < this.minPrice) {
        return
      }
      window.alert('你需要支付' + this.totalPrice + '元')
    }
  },
  transitions: {
    drop: {
      beforeEnter (el) {
        let count = this.balls.length
        while (count--) {
          let ball = this.balls[count]
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect()
            let x = rect.left - 32
            let y = -(window.innerHeight - rect.top - 22)
            el.style.display = ''
            el.style.webkitTransform = `translate3d(0,${y}px,0)`
            el.style.transform = `translate3d(0,${y}px,0)`
            let inner = el.getElementsByClassName('inner-hook')[0]
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`
            inner.style.transform = `translate3d(${x}px,0,0)`
          }
        }
      },
      enter (el) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)'
          el.style.transform = 'translate3d(0,0,0)'
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = 'translate3d(0,0,0)'
          inner.style.transform = 'translate3d(0,0,0)'
        })
      },
      afterEnter (el) {
        let ball = this.dropBalls.shift()
        if (ball) {
          ball.show = false
          el.style.display = 'none'
        }
      }
    }
  },
  components: {
    cartcontrol
  }

}
</script>
<style lang='stylus' rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      font-size: 0
      color: rgba(255, 255, 255, 0.4)
      .content-left
        flex: 1
        background: rgba(0 , 0, 0, 0.76)
        .logo-wrapper
          display: inline-block
          position: relative
          top: -10px
          margin: 0 12px
          padding: 6px
          height: 56px
          width: 56px
          box-sizing: border-box
          vertical-align: top
          border-radius: 50%
          background: rgb(58, 58, 59)
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            text-align: center
            background: rgb(51, 51, 51)
            &.highlight
              background: rgb(46, 141, 233)
            .icon-shopping_cart
              color: rgb(95, 95 , 99)
              line-height: 44px
              font-size: 30px
              &.highlight
                color: #fff
          .num
            position: absolute
            top: 0
            right: 0
            height: 11px
            line-height: 11px
            padding: 0 6px
            text-align: center
            border-radius: 18px
            font-size: 9px
            font-weight: 700
            color: #fff
            background: rgb(240, 20, 20)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          line-height: 24px
          color: rgba(255, 255, 255, 0.4)
          box-sizing: border-box
          font-size:20px
          font-weight: 700
          padding-right: 12px
          &.highlight
            color: #fff
            margin-top: 6px
        .desc
          position: absolute
          bottom: 10px
          left: 84px
          display: inline-block
          color: #fff
          font-size: 10px
          height: 10px
          line-height: 10px


      .content-right
        flex: 0 0 105px
        width: 105px
        background: rgb(82, 83, 85)
        .pay
          height: 48px
          font-weight: 700
          line-height: 48px
          text-align: center
          font-size: 16px
          &.enough
            background: rgb(56, 202, 115)
            color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        &.drop-transition
          transition: all 0.4s
          transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
          .inner
            width: 16px
            height: 16px
            border-radius: 50%
            background: rgb(49, 144, 232)
            transition: all 0.4s linear
    .shopcart-list
      position: absolute
      top: 0
      left: 0
      z-index: -1
      width: 100%
      &.fold-transition
        transition: all 0.5s
        transform: translate3d(0, -100%, 0)
      &.fold-enter, &.fold-leave
        transform: translate3d(0, 0, 0)
      .list-header
        height: 40px
        line-height: 40px
        padding: 0 18px
        background: #f3f5f7
        border-bottom: 1px solid rgba(7, 17, 27, 0.1)
        .name
          float: left
          font-size: 14px
          height: 14px
          line-height: 14px
          margin-top: 14px
          border-left:3px solid #2e8de9
          color: rgb(7, 17, 27)
        .empty
          float: right
          font-size: 12px
          color: rgb(0, 160, 220)
      .list-content
        padding: 0 18px
        max-height: 317px
        overflow: hidden
        background: #fff
        b
        .food
          position: relative
          padding: 12px 0
          box-sizing: border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            display: inline-block
            color: rgb(7, 17, 27)
            font-size: 16px
            line-height: 24px
          .price
            position: absolute
            right: 90px
            bottom: 12px
            line-height: 24px
            font-size: 16px
            font-weight: 700
            color: rgb(240, 20, 20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px
  .list-mask
    position: fixed
    top: 0
    left: 0
    height: 100%
    width: 100%
    z-index: 40
    &.fade-transition
      background: rgba(7, 17, 27, 0.4)
      opacity: 1
    &.fade-enter, &.fade-leave
      backgounf: rgb(7, 17, 27)
      opacity: 0

</style>