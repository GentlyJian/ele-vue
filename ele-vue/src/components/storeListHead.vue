<template>
  <section id="storeListHead" class="store-header">
    <div :style="{ top: offsetTop / 37.5 + 'rem' }" :class="{ fixed: fixTop }">
      <div class="filter-header">
        <a class="filter-nav" :class="[showSortFlag ? 'c3190e8' : '']" @click="showSort">综合排序</a>
        <a class="filter-nav" @click="handleCheck">距离最近</a>
        <a class="filter-nav">品质联盟</a>
        <a class="filter-nav" :class="[showScreeing ? 'c3190e8' : '']" @click="showScre">筛选</a>
      </div>
      <!-- 综合排序列表 -->
      <section class="sort" :class="{ open: showSortFlag }">
        <ul class="item-list">
          <li
            v-for="(item, index) in sortItems"
            :key="index"
            :style="{color: index == currentIndex ? '#3190E8' : '#666'}"
            class="item flex justify_between"
            @click="selectOrderType(index)"
          >
            <span>{{ item }}</span>
            <i class="iconfont">&#xe650;</i>
          </li>
        </ul>
      </section>
      <!-- 筛选列表 -->
      <section class="screening" :class="{ 'screening-open': showScreeing }">
        <div class="service">
          <!-- 商家服务 -->
          <div class="shop-service">
            <p class="title">商家服务（可多选）</p>
            <cube-checker v-model="checkerValue" :options="shopService">
              <cube-checker-item
                v-for="item in shopService"
                :key="item.value"
                :option="item"
                class="test"
              >
                <span class="item">
                  <!-- eslint-disable-next-line vue/no-v-html -->
                  <i class="iconfont" :style="{ color: item.color }" v-html="item.icon"></i>
                  {{ item.text }}
                </span>
              </cube-checker-item>
            </cube-checker>
          </div>
          <!-- 优惠活动 -->
          <div class="favo-activity">
            <p class="title">优惠活动（单选）</p>
            <ul class="flex wrap">
              <li
                v-for="(item, index) in favourableAct"
                :key="index"
                class="item"
                :class="[actIndex == index ? 'item_active' : '']"
                @click="actIndex = index"
              >
                {{ item.value }}
              </li>
            </ul>
          </div>
          <!-- 人均消费 -->
          <div class="per-consump">
            <p class="title">人均消费</p>
            <ul class="flex wrap">
              <li
                v-for="(item, index) in perConsump"
                :key="index"
                class="item"
                :class="[perIndex == index ? 'item_active' : '']"
                @click="perIndex = index"
              >
                {{ item.value }}
              </li>
            </ul>
          </div>
        </div>
        <div class="btn flex">
          <div class="btn1" @click="handleEmpty">清空</div>
          <div class="btn2" @click="showScre">确定</div>
        </div>
      </section>
      <div v-if="showSortFlag || showScreeing" class="mask"></div>
    </div>
  </section>
</template>

<script>
const SELECT_ORDER_TYPE = 'selectOrderType'

export default {
  props: {
    // 是否需要吸顶
    needFixTop: {
      type: Boolean,
      required: false,
      default: true
    },
    // 偏移值，用于多重吸顶的时候位置调整
    offsetTop: {
      type: [Number, String],
      required: false,
      default: 0
    }
  },
  data() {
    return {
      showSortFlag: false,
      currentIndex: 0,
      sortItems: ['综合排序', '好评优先', '起送价最低', '配送最快'],
      // 商家服务
      shopService: [
        {
          value: 1,
          text: '蜂鸟专送',
          icon: '&#xe601;',
          color: '#02ABE9'
        },
        {
          value: 2,
          text: '到点自取',
          icon: '&#xe652;',
          color: '#F89577'
        },
        {
          value: 3,
          text: '品牌商家',
          icon: '&#xe62d;',
          color: '#FE9736'
        },
        {
          value: 4,
          text: '新店',
          icon: '&#xe60f;',
          color: '#F6AD2A'
        },
        {
          value: 5,
          text: '接受预约',
          icon: '&#xe60f;',
          color: '#9C8EEF'
        },
        {
          value: 6,
          text: '食无忧',
          icon: '&#xe600;',
          color: '#FE869F'
        },
        {
          value: 7,
          text: '开发票',
          icon: '&#xe6cb;',
          color: '#96CB59'
        }
      ],
      // 优惠活动
      favourableAct: [
        {
          key: 1,
          value: '首单立减'
        },
        {
          key: 2,
          value: '门店新客立减'
        },
        {
          key: 3,
          value: '满减优惠'
        },
        {
          key: 4,
          value: '下单返红包'
        },
        {
          key: 5,
          value: '进店领红包'
        },
        {
          key: 6,
          value: '配送费优惠'
        },
        {
          key: 7,
          value: '赠品优惠'
        },
        {
          key: 8,
          value: '特价商品'
        },
        {
          key: 9,
          value: '品质联盟红包'
        }
      ],
      // 人均消费
      perConsump: [
        {
          key: 1,
          value: '￥20以下'
        },
        {
          key: 3,
          value: '￥20-￥40'
        },
        {
          key: 4,
          value: '￥40-￥60'
        },
        {
          key: 5,
          value: '￥60-￥80'
        },
        {
          key: 6,
          value: '￥80-￥100'
        },
        {
          key: 7,
          value: '￥100以上'
        }
      ],
      // 选中商家服务
      checkerValue: [],
      // 选中优惠活动
      actIndex: -1,
      // 选中人均服务
      perIndex: -1,
      showScreeing: false,
      container: null,
      fixTop: false,
      scrollTop: 0,
      distance: 0,
      tempDistance: 0
    }
  },
  mounted() {
    this.initData()
  },
  activated() {
    this.initData()
  },
  deactivated() {
    if (this.needFixTop) {
      document.removeEventListener('scroll', this.handleCheck)
    }
  },
  destroyed() {
    if (this.needFixTop) {
      document.removeEventListener('scroll', this.handleCheck)
    }
  },
  methods: {
    handleEmpty() {
      this.checkerValue = []
      this.actIndex = -1
      this.perIndex = -1
    },
    setFixTop(boolean) {
      this.fixTop = boolean
    },
    selectOrderType(index) {
      this.currentIndex = index
      this.showSort()
      this.$emit(SELECT_ORDER_TYPE, index)
    },
    initData() {
      if (this.needFixTop) {
        this.container = document.getElementById('storeListHead')
        this.scrollTop = this.container.getBoundingClientRect().y
        document.addEventListener('scroll', this.handleCheck)
        this.$nextTick(() => {
          window.scrollBy(0, this.tempDistance)
        })
      }
    },
    handleCheck() {
      this.checkFix(this.container)
    },
    checkFix(container) {
      const { top, y } = container.getBoundingClientRect()
      this.distance = top || y || 0
      this.tempDistance = document.documentElement.scrollTop
      if (this.distance >= this.offsetTop) {
        this.fixTop = false
      } else {
        this.fixTop = true
      }
    },
    // 排序
    showSort() {
      this.showSortFlag = !this.showSortFlag
      this.showScreeing = false
      if (this.showSortFlag) {
        document.body.classList.add('hide')
      } else {
        document.body.classList.remove('hide')
      }
      if (!this.needFixTop) {
        return
      }
      if (!this.fixTop) {
        this.fixTop = true
        window.scrollTo({
          top: this.scrollTop
        })
      }
    },
    // 筛选
    showScre() {
      this.showScreeing = !this.showScreeing
      this.showSortFlag = false
      if (this.showScreeing) {
        document.body.classList.add('hide')
      } else {
        document.body.classList.remove('hide')
      }
      if (!this.needFixTop) {
        return
      }
      if (!this.fixTop) {
        this.fixTop = true
        window.scrollTo({
          top: this.scrollTop
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.store-header {
  position: relative;
  height: 40px;
  .filter-header {
    display: flex;
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    z-index: 4;
    height: 100%;
    line-height: 40px;
    background-color: #fff;
    .filter-nav {
      flex: 1;
      text-align: center;
      color: #666;
      position: relative;
      font-size: 0.373333rem;
      z-index: 101;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
}

.sort {
  width: 100%;
  background-color: #ffffff;
  position: absolute;
  top: 40px;
  z-index: 4;
  max-height: 0px;
  overflow: auto;
  opacity: 0;
  transition: all 0.2s ease-in-out;
  .item {
    font-size: 14px;
    color: black;
    padding: 0px 20px;
    line-height: 40px;
  }
}
.open {
  opacity: 1;
  max-height: 250px;
}
.mask {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: 3;
  background: rgba(0, 0, 0, 0.5);
  transition: all 0.3s ease-in-out;
}
.screening {
  width: 100%;
  background-color: #ffffff;
  position: absolute;
  top: 40px;
  z-index: 4;
  max-height: 0px;
  overflow: auto;
  opacity: 0;
  transition: all 0.2s ease-in-out;
  position: relative;
  .title {
    font-size: 14px;
    margin-bottom: 20px;
  }
  .item {
    width: 30%;
    font-size: 13px;
    text-align: center;
    line-height: 40px;
    margin-right: 10px;
    margin-bottom: 10px;
    background: #fafafa;
    &:last-child {
      margin-right: 0;
    }
  }
  .item_active {
    font-weight: 700;
    color: #3190e8;
    background-color: #edf5ff;
    font-weight: 700;
    border: 0px;
    color: #3190e8;
  }
  .service {
    padding: 10px 15px;
  }
  .btn {
    line-height: 1.146667rem;
    width: 100%;
    .btn1 {
      flex: 1;
      background-color: #fff;
      color: #ddd;
      text-align: center;
    }
    .btn2 {
      flex: 1;
      color: #fff;
      background-color: #00d762;
      border: 0.013333rem solid #00d762;
      text-align: center;
    }
  }
}
.shop-service {
  .title {
    font-size: 0.324324rem;
    margin-bottom: 20px;
  }
}
.screening-open {
  opacity: 1;
  max-height: 350px;
}
.fixed {
  width: 100%;
  height: 40px;
  position: fixed;
  z-index: 101;
  background: #fff;
}
.c3190e8 {
  color: #3190e8 !important;
}
</style>

<style lang="scss">
.screening {
  .cube-checker-item {
    height: 40px;
    width: 30%;
    padding: 0;
    line-height: 40px;
    margin-right: 10px;
    margin-bottom: 10px;
    border: 0px;
    background: #fafafa;
    &:last-child {
      margin-right: 0;
    }
  }
  .cube-checker-item_active {
    font-weight: 700;
    color: #3190e8;
    background-color: #edf5ff;
    font-weight: 700;
    border: 0px;
    color: #3190e8;
  }
  .cube-checker-item_active::after {
    border: 0px;
  }
}
.hide {
  overflow: hidden;
}
</style>
