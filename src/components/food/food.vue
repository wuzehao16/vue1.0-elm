<template>
  <div class="food" v-show="showFlag" transition="move" v-el:food>
  	<div class="foot-content">
  		<div class="img-header">
  			<img :src="food.image">
  			<div class="back">
  				<i class="icon-arrow_lift" @click="hide"></i>
  			</div>
  		</div>
  		<div class="content">
  			<h1 class="title">{{food.name}}</h1>
  			<div class="count">月售{{food.sellCount}}份</div>
  			<div class="price">
			<span class="now">￥{{food.price}}</span>
			<span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
		    </div>
		    <div class="cartcontrol-wrapper">
  			<cartcontrol :food="food" v-show="food.count>0" v-ref:cartcontrol></cartcontrol>
  			</div>
  			<div class="buy" v-show="!food.count || food.count===0" @click.stop.prevent="addFirst(food,$event)" transition="fade">加入购物车</div>
  		</div>
  		<split v-show="food.info"></split>
  		<div class="info" v-show="food.info">
  			<div class="title">商品信息</div>
  			<p class="text">{{food.info}}</p>
  		</div>
  		<split></split>
  		<div class="rating">
  			<h1 class="title">商品评价</h1>
  			<ratingselect :select-type="selectType" :only-content="onlyContent"
  			:desc="desc" :ratings="food.ratings"></ratingselect>
  			<div class="rating-wrapper">
  				<ul v-show="food.ratings && food.ratings.length">
  					<li v-for="rating in food.ratings" class="rating-item" v-show="needShow(rating.rateType,rating.text)">
  						<div class="user">
  							<span class="name">{{rating.username}}</span>
  							<img :src="rating.avatar" height="12" width="12" class="avatar">
  						</div>
  						<div class="time">{{rating.rateTime | formatDate}}</div>
  						<p class="text">
  							<span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
  						</p>
  					</li >
  				</ul>
  				<div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
  			</div>	
  		</div>
  		<split></split>
  	</div>
  </div>
</template>
<script type='text/ecmascript-6'>
import BScroll from 'better-scroll'
import cartcontrol from '../cartcontrol/cartcontrol'
import {formatDate} from 'common/js/date'
import split from '../split/split'
import ratingselect from '../ratingselect/ratingselect'

const ALL = 2

export default {
  props: {
    food: {
      type: Object
    }
  },
  data () {
    return {
      showFlag: false,
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    }
  },
  methods: {
    show () {
      this.showFlag = true
      this.selectType = ALL
      this.onlyContent = false
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$els.food, {
            click: true
          })
        } else {
          this.scroll.refresh()
        }
      })
    },
    hide () {
      this.showFlag = false
    },
    addFirst (food, event) {
      if (!event._constructed) {
        return
      }
      console.log(this.$refs.cartcontrol)
      this.$refs.cartcontrol.addCart(event)
    },
    needShow (type, text) {
      if (this.onlyContent && !text) {
        return false
      }
      if (this.selectType === ALL) {
        return true
      } else {
        return type === this.selectType
      }
    }
  },
  events: {
    'ratingtype.select' (type) {
      this.selectType = type
      console.log(2)
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    },
    'content.toggle' (onlyContent) {
      console.log(3)
      this.onlyContent = onlyContent
      this.$nextTick(() => {
        this.scroll.refresh()
      })
    }
  },
  filters: {
    formatDate (time) {
      let date = new Date(time)
      return formatDate(date, 'yyyy-MM-dd hh:mm')
    }
  },
  components: {
    cartcontrol,
    split,
    ratingselect
  }
}
</script>
<style lang='stylus' rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    width: 100%
    background: #fff
    &.move-transition
      transition: all 0.2s linear
      transform: translate3d(0, 0, 0)
    &.move-enter, &.move-leave
      transform: translate3d(100%, 0, 0)
    .img-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100%
      img
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
      .back
        position: absolute
        top 10px
        left: 5px
        .icon-arrow_lift
          display: block
          padding: 6px
          color: #fff
    .content
      position: relative
      margin: 18px
      .title
        font-size: 14px
        font-weight: 500
        line-height: 14px
      .count
        margin:8px 0 10px 0
        color: rgb(147, 153, 159)
        line-height: 10px
        font-size:10px
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
        position: absolute
        right: 0px
        bottom: 0px
      .buy
        position:absolute
        right: 0px
        bottom:0px
        background: #3190e8
        padding: 5px 10px
        border-radius: 20px
        color: #fff
        font-size:12px


        margin-bottom: 6px
        &.fade-transition
          transition: all 0.2s
          opacity: 1
        &.fade-enter, &.fade-leave
          opacity: 0
    .info
      padding: 18px 0 18px 18px
      .title
        line-height: 14px
        padding-bottom: 10px
        font-size: 14px
        color: rgb(7, 17, 27)
        border-1px(rgba(7, 17, 27, 0.1))
      .text
        line-height: 24px
        font-size: 12px
        padding: 10px 8px 0 0
        color: rgb(147, 153, 159)
    .rating
      .title
        padding:18px 0 0 18px
        font-size: 14px
      .rating-wrapper
        padding: 0 18px
        .rating-item
          position: relative
          padding: 16px 0
          border-1px(rgba(7, 17, 27, 0.1))
          .user
            position: absolute
            right: 0
            top: 16px
            line-height: 12px
            font-size: 0
            .name
              display: inline-block
              margin-right: 6px
              vertical-align: top
              font-size: 10px
              color: rgb(147, 153, 159)
            .avatar
              border-radius: 50%
          .time
            margin-bottom: 6px
            line-height: 12px
            font-size: 10px
            color: rgb(147, 153, 159)
          .text
            line-height: 16px
            font-size: 12px
            color: rgb(7, 17, 27)
            .icon-thumb_up, .icon-thumb_down
              margin-right: 4px
              line-height: 16px
              font-size: 12px
            .icon-thumb_up
              color: rgb(0, 160, 220)
            .icon-thumb_down
              color: rgb(147, 153, 159)

        .no-rating
          padding: 16px 0
          font-size: 12px
          color: rgb(147, 153, 159)
</style>
