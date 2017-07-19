<template>
  <div >
     <div class="calendarWrap">
        <div class="month-detail">
            <ul>
                <li class="arrow-detail arrow-detail-left" @click="pickPre(currentYear,currentMonth)"><i class="arrow-left"></i></li>
                <li class="year-month" @click="pickYear(currentYear,currentMonth)">
                    <span class="choose-year">{{ currentYear }}年</span>
                    <span class="choose-month">{{ currentMonth }}月</span>
                </li>
                <li class="arrow-detail arrow-detail-right" @click="pickNext(currentYear,currentMonth)"><i class="arrow-right"></i></li>
            </ul>
        </div>
        <div id="calendar">
            <ul class="weekdays-detail">
                <li style="">日</li>
                <li>一</li>
                <li>二</li>
                <li>三</li>
                <li>四</li>
                <li>五</li>
                <li style="">六</li>
            </ul>
            <ul class="days-detail">
                <li @click="pick(day,$event)" v-for="(day,index) in days">
                    <p v-if="day.date.getMonth()+1 != currentMonth" class="other-month">
                        <span>{{ day.date.getDate() }}</span>
                        <span class="price" :class="{ useless: day.status == 0}" >
                            <b class="num-price" v-if="day.stock!=null">&yen;{{day.stock}}</b>
                            <b class="num-count" v-if="day.stock!=null">余{{day.count}}</b>
                        </span>
                    </p>
                    <template v-else>
                        <p  v-if="day.date.getFullYear() == new Date().getFullYear() && day.date.getMonth() == new Date().getMonth() && day.date.getDate() == new Date().getDate()">
                            <span>
                               <span :class="{ unable: day.stock==null && day.count==null}">今天</span>
                                <span class="price clearfix" :class="{ useless: day.status == 0}" >
                                    <b class="num-price" v-if="day.stock!=null">&yen;{{day.stock}}</b>
                                    <b class="num-count" v-if="day.stock!=null">余{{day.count}}</b>
                                </span>
                            </span>
                         </p>
                        <p v-else  :class="{ noneBorder: day.date.getDate() == 1,active : index == firstDayIndex[0]}">
                            <span >
                               <span :class="{ unable: day.stock==null && day.count==null }">{{ day.date.getDate() }}</span>
                                <span class="price" :class="{ useless: day.status == 0}">
                                    <b class="num-price" v-if="day.stock!=null">&yen;{{day.stock}}</b>
                                    <b class="num-count" v-if="day.stock!=null">余{{day.count}}</b>
                                </span>
                             </span>
                        </p>
                    </template>
                </li>
            </ul>
        </div>
    </div>
  </div>
</template>
<script>
	module.exports ={
		props:[
			'dateDataFromFather','id','city_id','dateAxios'
        ],
		mounted: function(){
			this.$set(this.$data,"price",this.dateDataFromFather);
			this.initData(this.dateAxios);
		},
		data: function(){
			return{
				currentDay: 1,
				currentMonth: 1,
				currentYear: 1970,
				currentWeek: 1,
				days: [],
				price: [],
				index: null,
				priceid: '',
				sdate: '',
				edate: '',
                firstDayIndex:[]
            }
		},
        watch:{
	        dateDataFromFather:function () {
		        this.$set(this.$data,"price",this.dateDataFromFather);
		        this.initData(this.dateAxios);
	        },
	        price:function(v){
		        for (var i = 0; i < this.price.length; i++) {
			        for (var j = 0; j < this.days.length; j++) {
				        var D = this.days[j].date.getDate();
				        var Y = this.days[j].date.getFullYear();
				        var M = this.days[j].date.getMonth() + 1;
				        var DD = this.formatDate(Y, M, D);
				        if (DD === this.price[i].date) {
					        this.days[j].stock = this.price[i].stock;
					        this.days[j].count = this.price[i].count;
					        this.days[j].status = this.price[i].status;
					        this.firstDayIndex.push(j)
				        }
			        }
		        }
	        }
        },
		methods: {
			initData: function(cur){
				var date;
				if (cur) {
					date = new Date(cur);
				} else {
					date = new Date();
					date.setDate(1);
				}
				this.currentDay = date.getDate();
				this.currentYear = date.getFullYear();
				this.currentMonth = date.getMonth() + 1;
				this.currentWeek = date.getDay(); // 1...6,0
				if (this.currentWeek == 0) {
					this.currentWeek = 7;
				}
				var str = this.formatDate(this.currentYear, this.currentMonth, this.currentDay);
				this.days.length = 0;

				// 今天是周日，放在第一行第7个位置，前面6个
				for (var i = this.currentWeek ; i >= 0; i--) {
					var d = new Date(str);
					d.setDate(d.getDate() - i);
					this.days.push({date: d, stock: null, count:null,status:null});

				}
				for (var i = 1; i <= 41 - this.currentWeek; i++) {
					var d = new Date(str);
					d.setDate(d.getDate() + i);
					this.days.push({date: d, stock: null,count:null,status:null});

				}
			},
			pick:function(day,event){
//				alert(day.date.getDate())
				this.$emit('changedatedata',{day,event});
			},
			pickPre:function(year, month){
				var d = new Date(this.formatDate(year, month, 1));
				d.setDate(0);
				this.initData(this.formatDate(d.getFullYear(), d.getMonth() + 1, 1));
				this.getPriceList();
				$(this.$el).find(".days-detail li p").removeClass('active');
			},
			pickNext:function(year, month){
				var d = new Date(this.formatDate(year, month, 1));
				d.setDate(35);
				this.initData(this.formatDate(d.getFullYear(), d.getMonth() + 1, 1));
				this.getPriceList();
				$(this.$el).find(".days-detail li p").removeClass('active');
			},
			pickYear:function(year, month){

			},
			// 返回 类似 2016-01-02 格式的字符串
			formatDate:function(year, month, day){
				var y = year;
				var m = month;
				if (m < 10) m = "0" + m;
				var d = day;
				if (d < 10) d = "0" + d;
				return y + "-" + m + "-" + d
			},
			getPriceList:function(){
				var y = this.currentYear,that = this;
				var m = this.currentMonth < 10 ? ('0' + this.currentMonth) : this.currentMonth;
				var dayZone = y + '-' + m;
				axios.get('/product-price/get-pricelist?date=' + dayZone + '&product_id='+ this.id+'&city_id='+this.city_id)
					.then(function (res) {
						var res_ = res.data,data = res_.data,newPriceList = [];
						if(res_.status == 1){
							var dataTopropItem = {};
							for(var key in data){
								if(data[key].count != null || data[key].price != null){
									dataTopropItem = {
										"date":data[key].data,
										"stock":data[key].price,
										"count":data[key].count,
										'status':data[key].istatus
									};
									newPriceList.push(dataTopropItem);
								}
							}
						}
						that.$set(that.$data,'price',newPriceList);
						that.$emit('changelimit',newPriceList);
					}).catch(function (err) {
					console.log(err);
				})
			}
		}
    }
</script>
<style scoped>
    * {
        box-sizing: border-box;
    }
    .calendarWrap .days-detail li > p.noneBorder{
        border-left: 1px solid #ccc;
        margin-left: -1px;
    }
    .clearfix:after{content:".";display:block;height:0;clear:both;visibility:hidden}
    .calendarWrap{
        position: relative;
        width: 500px;
    }
    ul {
        list-style-type: none;
    }

    body {
        font-family: Verdana, sans-serif;
        background: #E8F0F3;
    }

    .calendarWrap #calendar {
        height: 100%;
        width: 500px;
        border-right: 1px solid #e4e4e4;
        border-left: 1px solid #e4e4e4;
    }

    .calendarWrap .month-detail {
        width: 100%;
        height: 35px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        background: #00c8af;
    }

    .calendarWrap .month-detail ul {
        margin: 0;
        padding: 0;
        display: flex;
        height: 100%;
        flex-direction: row;
        width: 100%;
    }

    .calendarWrap .month-detail ul li {
        color: #b6b6b6;
        font-size: 16px;
        text-transform: uppercase;
        letter-spacing: 3px;
        line-height: 35px;
        flex: 1;
        text-align: center;
    }
    .calendarWrap .month-detail ul li.arrow-detail-left{
        text-align: right;
    }
    .calendarWrap .month-detail ul li.arrow-detail-right{
        text-align: left;
    }
    .calendarWrap .year-month {

    }

    .calendarWrap .year-month:hover {
        /*background: rgba(150, 2, 12, 0.1);*/
    }

    .calendarWrap .choose-year,.choose-month{
        text-align: center;
        cursor: pointer;
        color: #fff;
    }


    .calendarWrap .arrow-detail i{
        display: inline-block;
        width: 15px;
        height: 15px;
        border-left: 2px solid hsla(0,0%,100%,.6);
        border-bottom: 2px solid hsla(0,0%,100%,.6);
        transition: border .2s;
    }
    .calendarWrap .arrow-detail .arrow-left{
        transform: rotate(45deg);
        text-align: right;
    }
    .calendarWrap .arrow-detail .arrow-right{
        transform: rotate(-135deg);
        text-align: left;
    }
    .calendarWrap  .arrow-detail:hover {
        /*background: rgba(100, 2, 12, 0.1);*/
        cursor: pointer;
    }


    .calendarWrap .weekdays-detail {
        margin: 0;
        display: flex;
        flex-wrap: wrap;
        color: #999;
        justify-content: space-around;
        height: 28px;
        background: #f7f7f7;
        line-height: 28px;
    }

    .calendarWrap .weekdays-detail li {
        display: inline-block;
        width: 13.6%;
        text-align: center;
    }

    .calendarWrap .days-detail {
        padding-right: 0;
        padding-bottom: 0;
        background: #FFFFFF;
        margin: 0;
        display: flex;
        flex-wrap: wrap;
        justify-content: space-around;
        height: 100%;
    }

    .calendarWrap .days-detail li {
        list-style-type: none;
        display: inline-block;
        width: 14.2%;
        text-align: center;
        font-size: 12px;
        color: #000;
        -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
        height: 55px;
    }

    .calendarWrap .days-detail li:hover p{
        /*background: #fffbd3;*/
        cursor: pointer;
    }

    .calendarWrap .days-detail li > p {
        margin: 0;
        height: 100%;
        padding: 4px;
        border-right: 1px solid #ccc;
        border-bottom: 1px solid #ccc;
    }
    .calendarWrap .days-detail li:nth-child(7n) p{
       border-right:none ;
    }
    .calendarWrap .days-detail li  p.other-month{
        border-right: none;
        border-left: none;
    }
    /*.calendarWrap .days-detail li:last-child  p.other-month{
        border-left: 1px solid #ccc;
    }*/
    .calendarWrap .days-detail li .active{
        /*background: #e4f6ff;*/
    }


    .calendarWrap .days-detail li .other-month {
        color: #fff;
    }

    .calendarWrap .days-detail li .other-month .price b{
        color: #fff;
    }
    .calendarWrap .days-detail li .unable {
        color: gainsboro;
    }

    .calendarWrap .days-detail li p span {
        display: block;
        margin: 0 auto;
        width: 100%;
        text-align: left;
    }

    .calendarWrap .days-detail li p span.price {
        font-size: 12px;
    }
    .calendarWrap  .price b{
        display: block;
        /*margin-top: 0px;*/
        text-align: left;
        color: #00c8af;
        overflow: hidden;
        font-weight: 300;
    }
    .calendarWrap  .price.useless b{
        /*color: #fff;*/
        display: none;
    }

    .calendarWrap .price b.num-count{
       text-align: left;
        font-weight: 300;
        margin-top: -3px;
    }
</style>