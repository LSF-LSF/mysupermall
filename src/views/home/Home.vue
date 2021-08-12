<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control class="tabcontrol"
                 :titles="['流行','新款','精选']"
                 @tabclick="tabClick"
                 ref="tabcontrol1"
                 v-show="isTabFixed">

    </tab-control>

    <scroll class="content" ref="scroll" :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @loadMore="loadmore">


      <home-swiper :banners="banners" @swiperload="swiperload" ref="hSwiper"></home-swiper>
      <home-recommend :recommends="recommends"></home-recommend>
      <feature-view></feature-view>

      <tab-control
        :titles="['流行','新款','精选']"
        @tabclick="tabClick"
        ref="tabcontrol2"></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>

    <back-top @click.native="backClick" v-show="isShow"></back-top>
  </div>
</template>

<script>
import NavBar from "@/components/common/navbar/NavBar";
import Scroll from "@/components/common/scroll/Scroll";
import TabControl from "@/components/content/tabControl/TabControl";
import GoodsList from "@/components/content/goods/GoodsList";
import BackTop from "@/components/content/backtop/BackTop";


import HomeSwiper from "@/views/home/childrenComps/HomeSwiper";
import HomeRecommend from "@/views/home/childrenComps/HomeRecommend";
import FeatureView from "@/views/home/childrenComps/FeatureView";


import {getHomeMultidata, getHomeGoods} from "@/network/home";
import {debounce} from "@/common/utils";


export default {
  name: "Home",
  components: {
    NavBar,
    TabControl,
    GoodsList,
    Scroll,
    BackTop,


    HomeSwiper,
    HomeRecommend,
    FeatureView,

  },
  activated: function () {
    this.$refs.hSwiper.startTimer()
  },
  deactivated: function () {
    this.$refs.hSwiper.stopTimer()
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      },
      currentType: 'pop',
      isShow: false,
      tabOffSetTop: 0,
      isTabFixed: false
    }
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    }
  },
  mounted() {
    //1.图片加载的事件监听和防抖函数
    const refresh = debounce(this.$refs.scroll.refresh,)
    this.$bus.$on('itemImageLoad', () => {
      refresh()
    })
    //2获取tabControl的offsetTop
    //所有组件都有一个属性$el,用于获取组件的元素
  },
  created() {
    //1.请求多个数据
    this.getHomeMultidata();
    // 2.请求商品数据
    this.getHomeGoods('pop')
    this.getHomeGoods('new')
    this.getHomeGoods('sell')

  },
  methods: {
    //网络请求相关方法
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        // console.log(res)
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;

      })
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1

        ///完成加载更多
        this.$refs.scroll.finishPullUp()
      })
    },
    //事件监听相关方法
    tabClick(index) {

      switch (index) {
        case 0:
          this.currentType = 'pop'
          break
        case 1:
          this.currentType = 'new'
          break
        case 2:
          this.currentType = 'sell'
          break
      }
      this.$refs.tabcontrol1.currentIndex = index
      this.$refs.tabcontrol2.currentIndex = index
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0, 500)
    },
    contentScroll(position) {
      //1.判断backtop是否显示
      this.isShow = (-position.y) > 1000
      //2.判断tabcontrol是否吸顶
      // tabOffSetTop距离外部元素的位置
      this.isTabFixed = (-position.y) > this.tabOffSetTop
    },
    loadmore() {
      this.getHomeGoods(this.currentType)
    },
    swiperload() {
      this.tabOffSetTop = this.$refs.tabcontrol2.$el.offsetTop
    }

  }
}
</script>


// scoped只会对当前组件的样式起效果，避免class重复问题
// vh就是当前屏幕可见高度的1%
// 相对定位relative元素相对于原来位置偏移
// 绝对定位absolute在父元素没有设置相对定位或绝对定位的情况下，元素相对于根元素定位（即html元素）
// 如果是相对定位(relative),那么它的位置是保留的;
// 如果是绝对定位(absolute)或固定定位(fixed),则它的位置是不保留的。

<style scoped>
.home-nav {
  background-color: var(--color-tint);
  color: white;
  /*在使用浏览器原生滚动时，为了让导航一起滚动*/
  /*position: fixed;*/
  /*  left: 0;*/
  /*  top: 0;*/
  /*  right: 0;*/
  /*  z-index: 9;*/
}



#home {

  height: 100vh;
  position: relative;
}

/*.tabcontrol{*/
/*  position: sticky;*/
/*  top: 44px;*/
/* z-index 属性设置元素的堆叠顺序 */
/*  z-index: 9;*/
/*}*/
.content {
  /*height: calc(100% - 50px);*/
  /*overflow: hidden;*/
  overflow: hidden;
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;

}

.tabcontrol {
  position: relative;
  z-index: 9;
}

</style>
