<template>
    <div>
        <van-row type="flex" justify="center">
            <van-col span="23">
                <van-icon name="arrow-left" class="iconStyle" size="20px" @click="goBack"></van-icon>
                <van-cell-group>
                    <van-field v-model="tableId" label="餐桌编号" placeholder="请输入餐桌编号" @input="getTableId"/>
                    <van-cell title="预计上菜时间" center :value="order_detail.length*4+'分钟'"/>
                    <van-cell title="支付方式" center is-link :value="payStyle == 0 ? '支付宝' : '微信'" @click="showPayStyle"/>
                    <van-popup v-model="show" position="bottom" @click-overlay="closePopUp" @open="choosePayStyle">
                        <van-cell-group>
                            <van-cell is-link center @click="closePopUp">
                                <van-icon slot="right-icon" name="cross" size="20px"/>
                                <span slot="title" class="payFontStyle">选择支付方式</span>
                            </van-cell>
                            <van-cell is-link center @click="aliActive">
                                <van-icon slot="icon" name="alipay" color="#03a1e9" size="30px"/>
                                <van-icon slot="right-icon" name="checked" :color="aliColor" size="15px"/>
                                <span slot="title" class="chooseFontStyle">支付宝</span>
                            </van-cell>
                            <van-cell is-link center @click="weiXinActive">
                                <van-icon slot="icon" name="wechat" color="green" size="30px"/>
                                <van-icon slot="right-icon" name="checked" :color="weiXinColor" size="15px"/>
                                <span slot="title" class="chooseFontStyle">微信支付</span>
                            </van-cell>
                            <van-cell :border="otherCellshow" v-for="i in 8" :key="i"></van-cell>
                        </van-cell-group>
                    </van-popup>
                </van-cell-group>
                <div style="padding-top: 20px; padding-bottom: 10px; font-family: 'Microsoft YaHei'; ">
                    <van-cell title="菜品列表" center/>
                </div>
                <div v-for="(dishItem,index) in dishList" :key="index">
                    <div v-if="getDishNum(dishItem.dish_id)>0">
                        <van-card style="padding-top: 5px; padding-bottom: 5px; background-color: white; height: 50%"
                                  :num="getDishNum(dishItem.dish_id)"
                                  :price="dishItem.dish_price"
                                  :desc="dishItem.dish_description"
                        >
                            <div slot="title" class="dishTitle">
                                {{dishItem.dish_name}}
                            </div>
                            <div slot="thumb" style="width: 70%; height: 70%;">
                                <img class="imgStyle" :src="dishItem.dish_picture">
                            </div>
                        </van-card>
                    </div>
                </div>
                <van-cell>
                    <span slot="title">小计 <span
                            style="font-size: large">￥{{commdityPrice/100}}</span></span>
                </van-cell>
                <div style="padding-top: 20px">
                    <van-cell-group>
                        <van-field
                                v-model="message"
                                label="订单备注"
                                type="textarea"
                                placeholder="请输入口味、偏好"
                                rows="1"
                                autosize
                                @input="getFav"
                        />
                        <van-cell title="餐具份数" is-link :value="dishNumValue" @click="showDishNum"></van-cell>
                        <van-popup v-model="dishNum" position="bottom" @click-overlay="closeDishNum">
                            <van-picker
                                    show-toolbar
                                    title="餐具份数"
                                    :columns="columns"
                                    @cancel="closeDishNum"
                                    @confirm="updateDishNum"
                            />
                        </van-popup>
                        <van-cell title="发票信息" is-link value="未选择"></van-cell>
                    </van-cell-group>
                </div>
                <van-cell-group>
                    <van-cell :border="otherCellshow" v-for="i in 4" :key="i"></van-cell>
                </van-cell-group>
            </van-col>
        </van-row>
        <van-submit-bar
                :price="Number(commdityPrice)"
                button-text="确认支付"
                button-type="primary"
                @submit="confirmPay"
        >
        </van-submit-bar>
    </div>
</template>

<script>
    export default {
        name: "OrderInfo",
        data() {
            return {
                show: false,
                otherCellshow: false,
                aliColor: "grey",
                weiXinColor: "grey",
                payStyle: Number,
                dishNum: false,
                columns: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '更多'],
                dishNumValue: Number,
                tableId: String,
                message: String,
                order_detail: [],
                noUse: null,
                dishList: Array,
                commdityPrice: Number,
                temp:[]
            }
        },
        created() {
            this.dishList = JSON.parse(localStorage.getItem('dishList'));
            this.temp = JSON.parse(localStorage.getItem('user_dish_num'));
            this.commdityPrice = localStorage.getItem('commidity_price');
            if(localStorage.getItem('tableId') === null){
                this.tableId = '';
                localStorage.setItem('tableId','');
            } else{
                this.tableId = localStorage.getItem('tableId');
            }
            if(localStorage.getItem('message') === null){
                this.message = '';
                localStorage.setItem('message','');
            } else{
                this.message = localStorage.getItem('message');
            }
            if(localStorage.getItem('dishNumValue') === null){
                this.dishNumValue = 0;
                localStorage.setItem('dishNumValue','0');
            } else{
                this.dishNumValue = Number(localStorage.getItem('dishNumValue'));
            }
            if(localStorage.getItem('payStyle') === null){
                this.payStyle = 0;
                localStorage.setItem('payStyle','0');
            } else{
                this.payStyle = Number(localStorage.getItem('payStyle'));
            }
            for(let i in this.dishList){
                if(this.temp[this.dishList[i].dish_id] > 0){
                    this.order_detail.push({dish_id:this.dishList[i].dish_id,dish_num:this.temp[this.dishList[i].dish_id]});
                }
            }
        },
        destroyed(){
            this.clearInfo(0);
        },
        methods: {
            getDishNum(index) {
                return this.temp[index];
            },
            showPayStyle() {
                this.show = true;
            },
            closePopUp() {
                this.show = false;
            },
            showDishNum() {
                this.dishNum = true;
            },
            closeDishNum() {
                this.dishNum = false
            },
            updateDishNum(value) {
                this.dishNumValue = value;
                localStorage.setItem('dishNumValue',value);
                this.dishNum = false;
            },
            aliActive() {
                this.payStyle = 0;
                localStorage.setItem('payStyle','0');
                this.show = false;
            },
            weiXinActive() {
                this.payStyle = 1;
                localStorage.setItem('payStyle','1');
                this.show = false;
            },
            choosePayStyle() {
                if (this.payStyle === 0) {
                    this.aliColor = "green";
                    this.weiXinColor = "grey";
                } else {
                    this.aliColor = "grey";
                    this.weiXinColor = "green";
                }
            },
            goBack() {
                this.clearInfo(0);
                this.$router.push({name: 'dish'});
            },
            getFav(value) {
                this.message = value;
                localStorage.setItem('message',value);
            },
            confirmPay() {
                this.$axios({
                    method: 'post',
                    url: '/orders/create',
                    headers: {
                        Authorization: "Bearer " + localStorage.getItem('user_token')
                    },
                    data: {
                        order_table: Number(this.tableId),
                        order_price: Number(this.commdityPrice / 100),
                        order_script: this.message,
                        order_status: 1,
                        order_detail: this.order_detail,
                        table_ware_num: Number(this.dishNumValue),
                        pay_method: this.payStyle
                    }
                })
                    .then(response => {
                        this.clearInfo(1);
                        let len = response.data.pay_url.length;
                        // eslint-disable-next-line no-console
                        console.log(len);
                        // eslint-disable-next-line no-console
                        console.log(response.data.pay_url.substring(25,len));
                        localStorage.setItem('payUrl','http://geeking.tech:8008'+response.data.pay_url.substring(25,len));
                        this.$router.push({name: 'confirmPay'});
                    })
                    .catch(error => {
                        if (error.response.status === 401) {
                            this.$dialog.alert({
                                message: '登录已过期，请重新登录！'
                            });
                            localStorage.setItem('user_name','');
                            this.$router.push({name: 'login'});
                        } else if(error.response.status === 400){
                            this.$dialog.alert({
                                message: '请输入订单备注！'
                            });
                        }
                    })
            },
            getTableId(value) {
                this.tableId = value;
                localStorage.setItem('tableId',value);
            },
            clearInfo(index) {
                if(index === 0){
                    localStorage.removeItem('payStyle');
                    localStorage.removeItem('message');
                    localStorage.removeItem('dishNumValue');
                } else {
                    localStorage.removeItem('user_dish_num');
                    localStorage.removeItem('commidity_num');
                    localStorage.removeItem('commidity_price');
                    localStorage.removeItem('dishList');
                    localStorage.removeItem('chooseDish');
                    localStorage.removeItem('payStyle');
                    localStorage.removeItem('message');
                    localStorage.removeItem('dishNumValue');
                    localStorage.removeItem('tableId');
                }
            }
        },
    }
</script>

<style scoped>
    .payFontStyle {
        font-size: medium;
        font-family: "Microsoft YaHei", serif;
    }

    .chooseFontStyle {
        font-family: "Microsoft YaHei", serif;
        padding-left: 10px;
    }

    .iconStyle {
        padding-top: 20px;
        padding-bottom: 50px;
    }

    .dishTitle {
        font-size: 20px;
        font-style: normal;
        font-family: "Microsoft YaHei", serif;
        font-weight: bold;
    }

    .imgStyle {
        height: 100%;
        width: 100%;
        border-radius: 5px;
    }
</style>