<template>
  <div class="goods">
    <div class="menu-wrapper" v-el:menu-wrapper>
    	<ul>
    		<li v-for="item in goods" class="menu-items" :class="{'current':currentIndex===$index}" @click="selectMenu($index,$event)">
    			<span class="text">
    			  <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>
    				{{item.name}}
    			</span>
    		</li>
    	</ul>
    </div>
    <div class="foods-wrapper" v-el:food-wrapper>
    	<ul>
    		<li v-for="item in goods" class="food-list food-list-hook">
    			<h1 class="title">{{item.name}}</h1>
    			<ul>
    				<li @click="selectFood(food,$event)" v-for="food in item.foods" class="foot-item">
    					<div class="icon">
    						<img :src="food.icon" width="57" height="57">
    					</div>
    					<div class="content">
    						<h2 class="name">{{food.name}}</h2>
    						<p class="desc" v-show="food.description">{{food.description}}</p>
    						<div class="extra">
    							<span>月售{{food.sellCount}}份</span>
    							<span>好评率{{food.rating}}%</span>
    						</div>
    						<div class="price">
    							<span class="now">￥{{food.price}}</span>
    							<span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
    						</div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
    					</div>
    				</li>
    			</ul>
    		</li>
    	</ul>
    </div>
    <shopcart v-ref:shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :select-foods="selectFoods"></shopcart>
  </div>
  <food :food="selectedFood" v-ref:food></food>
</template>
<script type='text/ecmascript-6'>
import BScroll from 'better-scroll'
import shopcart from '../shopcart/shopcart'
import cartcontrol from '../cartcontrol/cartcontrol'
import food from '../food/food'
const ERR_OK = 0
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data () {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    }
  },
  computed: {
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i]
        let height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i
        }
      }
      return 0
    },
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  created () {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
    this.$http.get('/api/goods').then((res) => {
      res = res.body
      if (res.errno === ERR_OK) {
        this.goods = res.data
        this.$nextTick(() => {
          this._initScroll()
          this._calculateHeight()
        })
      }
    })
  },
  methods: {
    selectMenu (index, event) {
      if (!event._constructed) {
        return
      }
      let foodList = this.$els.foodWrapper.getElementsByClassName('food-list-hook')
      let el = foodList[index]
      this.foodsScroll.scrollToElement(el, 300)
    },
    _initScroll () {
      this.menuScroll = new BScroll(this.$els.menuWrapper, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$els.foodWrapper, {
        probeType: 3, mouseWheel: true, click: true
      })
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
      })
    },
    _calculateHeight () {
      let foodList = this.$els.foodWrapper.getElementsByClassName('food-list-hook')
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    },
    _drop (target) {
      this.$nextTick(() => {
        this.$refs.shopcart.drop(target)
      })
    },
    selectFood (food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.food.show()
    }
  },
  components: {
    shopcart,
    cartcontrol,
    food
  },
  events: {
    'cart.add' (target) {
      this._drop(target)
    }
  }
}
</script>
<style lang='stylus' rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 0
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-items
        display: table
        padding: 0 12px
        height: 54px
        width: 56px
        line-height: 14px
        &.current
          position: relative
          z-index: 10
          background: #fff
          border-left: 2px solid #0089dc
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          width: 12px
          height: 12px
          margin-right: 4px
          background-size: 12px 12px
          background-repeat: no-repeat
          vertical-align: top
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          text-align: center
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        background-color: #f3f5f7
        font-size: 12px
        line-height: 26px
        border-left: 2px solid #d9dde1
        color: rgb(147, 153, 159)
      .foot-item
        display: flex
        margin: 18px 0 18px 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          img
            border-radius: 2px
        .content
          flex: 1
          vertical-align: top
          padding: 2px 18px 0 10px
          font-size: 0
          .name
            font-size: 14px
            line-height: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            font-size: 10px
            line-height: 10px
            margin: 8px 0
            color: rgb(147, 153, 159)
          .extra
            span
              padding-right: 12px
          .price
            line-height: 24px
            font-weight: normal
            .now
              margin-right: 8px
              font-size: 18px
              color: #f74342
            .old
              text-decoration: line-through
              font-size: 12px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position:absolute
            right: 0
            bottom: 12px
</style>
