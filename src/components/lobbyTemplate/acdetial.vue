<template>
    <div id="pa_con">
        <header id="pa_head" class="new_header">
            <div class="left">
                <a href="javascript:;" @click.prevent="goBack">
                    <span class="icon icon_back"></span>
                </a>
            </div>
            <h2 class="center title_name">帐户明细</h2>
            <div class="right"></div>
        </header>
        <div id="pa_content">
            <div id="betting_record" class="tab_container tabBox ac_detial_area">
                <div class="hd">
                    <ul class="tab tab_mid">
                        <li :class="tab.active?'on':''" @click="selectTab($event, tab)" v-for="(tab,index) in tabs"><a href="javascript:;">{{tab.title}}</a></li>
                    </ul>
                </div>
                <div class="bd">
                    <ul class="tab_content">
                        <li :class="(day.active?'active ':'')+' slide_toggle bet_day ac_detial new_panel'" v-for="(day,index) in activeTab.days">
                            <div class="panel_title new_panel_top" @click="selectDayTab($event, day)">
                                <strong>{{day.pdate}}<!-- 09月14日 --></strong><span></span>
                            </div>
                            <ul class="panel new_panel_center">
                                <li class="bet_data ac_data" data-status="not_open" v-for="item in day.list">
                                    <router-link :to="{ name:'acDetailData', params:{ model:item, data:$data} }">
                                       <!-- <div class="prd_num"><span>{{dateFormat(item.createTime, 'yyyy/mm/dd HH:mm') || '1990/1/1 00:00'}}</span></div>-->
                                        
                                        <div class="item arrow_list_dark">
                                            <div class="badge">
                                                <span :class="'icon_account icon_' + (tradeTypeConfigItemGet(item).class || 'ac01')"></span>
                                            </div>
                                            <div class="lottery_t ssc" v-if="(item.dealType!='3')">
                                                <p>
                                                    {{ (item.dealType == '2') ? item.actionTypeName : tradeTypeConfigItemGet(item).name}}<label
                                                        :class="'sta '+ (statusConfig[item.state] && statusConfig[item.state].class)">{{item.stateName}}</label>
                                                </p>
                                                <span class="prd_num"><span>{{formatTimeUnlix(item.createTime,'0')}}</span></span>
                                                <strong>{{moneyType[item.dealType] || '-'}}<!-- 充值 -->: {{(item && formatNumber(roundAmt(item.tradeAmount))) || '0.00'}}</strong>
                                            </div>
                                            <div class="lottery_t ssc" v-if="(item.dealType=='3')">
                                                <p>{{ item.tradeTypeName}}<label
                                                        :class="'sta '+ (statusConfig[item.state] && statusConfig[item.state].class)">{{item.stateName}}</label>
                                                </p>
                                                <span class="prd_num"><span>{{formatTimeUnlix(item.createTime, '0')}}</span></span>
                                                <strong>{{item.dealTypeName}}<!-- 充值 -->
                                                    : {{(item && formatNumber(roundAmt(item.tradeAmount))) || '0.00'}}</strong>
                                            </div>
                                            <div class="icon icon_arrow_light"></div>
                                        </div>
                                    </router-link>
                                </li>
                                
                            </ul>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        <footer id="pa_foot"></footer>
        <div class="so-shade"></div>


    </div>
</template>

<script>
// import $ from "jquery";
import Mixin from '@/Mixin'
import FooterNav from '@/components/Footer'

export default {
    name: 'acdetial',
    mixins:[Mixin],
    components: {
        FooterNav ,
    },
    data: function() {
        return {
            moneyType:{
                '0':'充值',
                '1':'提款',
                '2':'优惠'
            },
            tradeTypeConfig:{
                '1':{ name:'公司入款', class:'ac03' },
                '3':{ name:'线上入款', class:''},
                '5':{ name:'系统入款', class:'ac01'},
                '7':{ name:'会员出款', class:'ac03'},
                '8':{ name:'系统提款', class:'ac02'},
            },
            actionTypeConfig:{
                '1':{ class:'ac03', name:'派奖' },
                '2':{ class:'ac01', name:'系统入款' },
                '3':{ class:'ac03', name:'公司入款' },
                '4':{ class:'ac02', name:'系统提款' },
                '5':{ class:'ac03', name:'会员出款' },
            },
            activeTab:{ value:1, days:[] }, //当前选项卡
            statusConfig:{
                // { '0':'未处理', '2':'未处理', '3':'失败', '4':'成功', '5':'未处理' }
                '0':{ name:'未处理',class:'sta02'},
                '2':{name:'未处理',class:'sta02'},
                '3':{name:'失败',class:'sta03'},
                '4':{name:'成功',class:'sta01'},
                '5':{name:'未处理',class:'sta02'}
            },
            dealType: '',
            itemShow1: true,

        }
    },
    computed:{
        tabs:function(){
            return this.$parent.$data.acDetail.tabs;
        }
    },
    mounted:function() {
        //1 为全部选项卡的值
        let tab = this.tabs.filter(tab=>{return tab.active})[0];
        this.loadTab(tab);
        //scrollTo(0,0); // 回到顶部
        document.documentElement.scrollTop = document.body.scrollTop=0; // 回到顶部
    },
    methods: {
        itemShow: function (item) {
            this.itemShow1 = ( item.dealType != '2' || item.dealType == '2' && item.stateName == '成功')
            return this.itemShow1
        },
        back1:function(){
            this.$router.back();
        },
        tradeTypeConfigItemGet:function(item){
            return this.tradeTypeConfig[item.tradeType] || {};
        },
        loadTab:function(tab){
           /* tab.days.length<=0 && this.getDays(tab.value).then(res=>{*/
             this.getDays(tab.value).then(res=>{
                if (!res.data){
                    tab.days = [];
                    return false;
                }
                tab.days = res.data.map((item,index)=>{
                    if (index == 0){
                        item.active = true; 
                    }else{
                        item.active = false; 
                    }
                    item.list = [];
                    return item;
                });
                const day = tab.days[0];
                if (day){
                    this.getList(tab.value, day.pdate).then(res=>{
                        day.list = res.data;
                        localStorage.setItem('model',JSON.stringify( day.list))
                    });
                }
            });
            this.activeTab = tab;
            console.log(tab)
        },
        selectDayTab:function(e, day){
            if (day.active){
                day.active = false
            }else{
                this.activeTab.days.forEach(item=>{item.active=false});
                day.active = true;
            }
            day.list.length<=0 && this.getList(this.activeTab.value, day.pdate).then(res=>{
                day.list = res.data;
                localStorage.setItem('model',JSON.stringify( day.list))
            });
        },
        selectTab:function(e, tab){
            // if (tab.active || tab.value == 4){
            //     return false;
            // }
            this.tabs.forEach(item=>{item.active=false});
            tab.active = true;
            this.loadTab(tab);
        },
        getDays:function(searchType){
            return new Promise((resolve, reject)=>{
                this.ajax({
                    url: this.action.forseti + 'api/pay/tradeDayList',
                    data: {searchType: searchType || 1, page:1, rows:10 }, // 1全部，2充值，3提现
                    success: (res) => {
                        resolve(res);
                    }
                });
            });
        },
        getList:function(searchType, pdate) {
            return new Promise((resolve, reject)=>{
                this.ajax({
                    url: this.action.forseti + 'api/pay/tradeList',
                    data: {searchType: searchType || 1, pdate:pdate, page:1, rows:50 }, // 1全部，2充值，3提现
                    success: (res) => {
                        resolve(res);
                    },
                });
            });

        },

    }

}
</script>
<style>
    .bet_day.active .panel { display: block;  }
</style>