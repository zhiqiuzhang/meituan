<template>
  <div class="goods">
  	<div class="menu-wrapper" ref="menuScroll">
  		<ul>
  			<!-- 专场 -->
    		<li 
    		class="menu-item" 
    		:class="{'current': currentIndex===0}" 
    		@click="selectMenu(0)">
    			<p class="text">
    				<img class="icon" :src="container.tag_icon" v-if="container.tag_icon" alt=""> 
    				{{container.tag_name}}
    			</p>
    		</li>

    		<li 
    		class="menu-item" 
    		:class="{'current':currentIndex === index+1}" 
    		v-for="(item, index) in goods" 
    		:key="index" 
    		@click="selectMenu(index+1)">
    			<p class="text">
    				<img class="icon" :src="item.icon" v-if="item.icon" alt=""> 
    				{{item.name}}
    			</p>
    			<i class="num" v-show="calculateCount(item.spus)">
    				{{ calculateCount(item.spus) }}
    			</i>
    		</li>
    	</ul>
  	</div>
	<!-- 商品分类 -->
  	<div class="foods-wrapper" ref="foodScroll">
  		<ul>
  			<!-- 专场 -->
  			<li class="container-list food-list-hook">
  				<div v-for="(item, index) in container.operation_source_list" :key="index">
  					<img :src="item.pic_url" alt="">
  				</div>
  			</li>
  			<!-- 具体分类 -->
  			<li v-for="(item, index) in goods" :key="index" class="food-list food-list-hook">
  				<h3 class="title">
  					{{ item.name }}
  				</h3>
  				<!-- 具体商品分类 -->
  				<ul>
            <li 
              v-for="(food,index) in item.spus" 
              :key="index" 
              @click="foodDetail(food)"
              class="food-item">
              <div class="icon" :style="head_bg(food.picture)"></div>
              <div class="content">
                <h3 class="name">{{food.name}}</h3>
                <p class="desc" v-if="food.description">{{food.description}}</p>
                <div class="extra">
                  <span class="saled">{{food.month_saled_content}}</span>
                  <span class="praise">{{food.praise_content}}</span>
                </div>
                <img class="product" :src="food.product_label_picture" alt="">
                <p class="price">
                  <span class="text">${{food.min_price}}</span>
                  <span class="unit">/{{food.unit}}</span>
                </p>
              </div>
              <div class="cartcontrol-wrapper">
                <app-cartcontroll :food="food"></app-cartcontroll>
              </div>
            </li>
          </ul>
  				
  			</li>
  		</ul>
  	</div>
  <!-- 	购物车 -->
    <app-cart :poiInfo='poiInfo' :selecFoods='selecFoods'></app-cart>

    <!-- 食品详情 -->
	<app-shopdetail :food="selecfood" ref="detailShow"></app-shopdetail>
  </div>
</template>

<script>

import Bscroll from 'better-scroll'
import ShopCart from '../shopcart/Shopcart'
import CartControll from '../cartcontroll/CartControll'
import ShopDetail from '../shopdetail/Shopdetail'
export default {
  name: 'Goods',
  components: {
    'app-cart': ShopCart,
    'app-cartcontroll': CartControll,
    'app-shopdetail': ShopDetail
  },
  data() {
  	return {
  		container: {},
  		goods: [],
  		poiInfo:{},
  		heightLength: [],
  		menuScroll: {},
  		foodScroll: {},
  		scrollY: 0,
  		selecfood: {}
  	}
  },
  computed: {
  	currentIndex() {
  		for(let i = 0; i <this.heightLength.length; i++ ) {
  			let height1 = this.heightLength[i];
  			let height2 = this.heightLength[i + 1];
  			if(!height2 || this.scrollY >= height1 && this.scrollY < height2) {
  				return i;
  			}
  		}
  		return 0;
  	},
  	selecFoods() {
  		let foods = [];
  		this.goods.forEach((myfoods) => {
  			myfoods.spus.forEach((foodcount) => {
  				if(foodcount.count > 0) {
  					foods.push(foodcount);
  				}
  			})
  		})
  		return foods;
  	}
  },
  methods: {
  	head_bg(imgName) {
      return "background-image: url(" + imgName + ");"
  	},
  	initScroll() {
  		this.menuScroll = new Bscroll(this.$refs.menuScroll, {click: true});
  		this.foodScroll = new Bscroll(this.$refs.foodScroll, {
  			probeType: 3,
  			click: true
  		});
  		this.foodScroll.on('scroll', (pos) => {
  			this.scrollY = Math.abs(Math.round(pos.y));
  			console.log(this.scrollY);
  		})
  	},
  	calculateHeight() {
  		let liList = this.$refs.foodScroll.getElementsByClassName('food-list-hook');
  		let height = 0;
  		this.heightLength.push(height);
  		for(let i = 0;i < liList.length; i++) {
  			let item = liList[i].clientHeight;
  			height += item;
  			this.heightLength.push(height);
  		}
  		console.log(this.heightLength);
  		// return heightLength;
  	},
  	selectMenu(index) {
  		// console.log(2);
  		let liList = this.$refs.foodScroll.getElementsByClassName('food-list-hook');
  		let element = liList[index];
  		this.foodScroll.scrollToElement(element, 250);
  	},
  	calculateCount(spus) {
  		let count = 0;
  		spus.forEach((food) => {
  			if(food.count > 0) {
  				count += food.count
  			}
  		})
  		return count;
  	},
  	foodDetail(food) {
  		this.selecfood = food;
  		this.$refs.detailShow.showView();
  	}
  	

  },
  created() {
  	fetch('/api/goods').then((res)=>{
  		return res.json();
  	}).then((res) => {
  		if(res.code == 0) {
  			this.container = res.data.container_operation_source;
        	this.goods = res.data.food_spu_tags;
        	this.poiInfo = res.data.poi_info;
        	this.$nextTick(() => {
        		this.initScroll();
        		this.calculateHeight();
        	})
        	
  		}
  	});
  }
}
</script>

<style scoped>

	.goods{
  display: flex;
  position: absolute;
  top: 190px;
  bottom: 51px;
  overflow: hidden;
  width: 100%;
}

.goods .menu-wrapper{
  flex:0 0 85px;
  background: #f4f4f4;
}

.goods .foods-wrapper{
  flex:1;
  /* background: blue; */
}

/* Menu item */ 
.goods .menu-wrapper .menu-item{
	padding: 16px 23px 15px 10px;
	border-bottom: 1px solid #E4E4E4;
  position: relative;
}

.goods .menu-wrapper .menu-item .text{
	font-size: 13px;
	color: #333333;
	line-height: 17px;
	vertical-align: middle;
	-webkit-line-clamp: 2;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}

.goods .menu-wrapper .menu-item .text .icon{
	width: 15px;
	height: 15px;
	vertical-align: middle;
}

/* 专场样式 */ 
.goods .foods-wrapper .container-list{
	padding: 11px 11px 0 11px;
	border-bottom: 1px solid #E4E4E4;
}

.goods .foods-wrapper .container-list img{
	width: 100%;
	margin-bottom: 11px;
	border-radius: 5px;
}

/* 具体分类商品布局 */ 
.goods .foods-wrapper .food-list{
	padding: 11px;
}



.goods .foods-wrapper .food-list .title{
	height: 13px;
	font-size: 13px;
	background: url(./img/btn_yellow_highlighted@2x.png) no-repeat left center;
	background-size: 2px 10px;
	padding-left: 7px;
	margin-bottom: 12px;
}




.goods .foods-wrapper .food-list .food-item{
	display: flex;
	margin-bottom: 25px;
  position: relative;
}

.goods .foods-wrapper .food-list .food-item  .icon{
	flex: 0 0 63px;
	background-position: center;
	background-size: 120% 100%;background-repeat: no-repeat;
	margin-right: 11px;
	height: 75px;
}
.goods .foods-wrapper .food-list .food-item .content{
	flex: 1;
}

/* 具体内容样式 */ 
.goods .foods-wrapper .food-list .food-item .content .name{
	font-size: 16px;
	line-height: 21px;
	color: #333333;
	margin-bottom: 10px;
	padding-right: 27px;
}

.goods .foods-wrapper .food-list .food-item .content .desc{
	font-size: 10px;
	line-height: 19px;
	color: #bfbfbf;
	margin-bottom: 8px;
	
	/* 超出部分显示省略号*/
	-webkit-line-clamp: 1;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}

.goods .foods-wrapper .food-list .food-item .content .extra{
	font-size: 10px;
	color: #BFBFBF;
	margin-bottom: 7px;
}
.goods .foods-wrapper .food-list .food-item .content .extra .saled{
	margin-right: 14px;
}
.goods .foods-wrapper .food-list .food-item .content .product{
	height: 15px;
	margin-bottom: 6px;
}
.goods .foods-wrapper .food-list .food-item .content .price{
	font-size: 0;
}
.goods .foods-wrapper .food-list .food-item .content .price .text{
	font-size: 14px;
	color: #fb4e44;
}
.goods .foods-wrapper .food-list .food-item .content .price .unit{
	font-size: 12px;
	color: #BFBFBF;
}
/* 当前选中 */ 
.goods .menu-wrapper .menu-item.current{
	background: white;
	font-weight: bold;
	margin-top: -1px;
}
.goods .menu-wrapper .menu-item:first-child.current{
	margin-top: 1px;
}

.goods .foods-wrapper .food-list .food-item .cartcontrol-wrapper{
	position: absolute;
	right: 0;
	bottom: 0;
}

.goods .menu-wrapper .menu-item .num{
	position: absolute;
	right: 5px;
	top: 5px;
	width: 13px;
	height: 13px;
	border-radius: 50%;
	color: white;
	background: red;
	text-align: center;
	font-size: 7px;
	line-height: 13px;
}
</style>