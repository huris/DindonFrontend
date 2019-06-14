<template>
    <div>
        <van-row type="flex" justify="center">
            <van-col span="23">
                <van-icon name="arrow-left" class="iconStyle" size="20px" @click="goBack"></van-icon>
                <van-cell-group>
                    <van-cell title="订单状态" center :value="orderStatus(orderDetail.transaction.order_status)"/>
                    <van-cell title="餐桌编号" center :value="orderDetail.order_table"/>
                    <van-cell title="支付方式" center :value="orderDetail.transaction.pay_method == 0 ? '支付宝' : '微信'"/>
                </van-cell-group>
                <div style="padding-top: 20px; padding-bottom: 10px; font-family: 'Microsoft YaHei',serif; ">
                    <van-cell title="菜品列表" center/>
                </div>
                <div v-for="(dishItem,index) in orderDetail.order_detail" :key="index">
                    <van-card style="padding-top: 5px; padding-bottom: 5px; background-color: white; height: 50%"
                              :num="dishItem.dish_num"
                              :price="dishItem.dish_price"
                              :desc="dishItem.dish_description"
                    >
                        <div slot="title" class="dishTitle">
                            {{dishItem.dish_name}}
                        </div>
                        <div slot="thumb" style="width: 70%; height: 70%;">
                            <img class="imgStyle" :src="dishItem.dish_picture" alt="dish_picture">
                        </div>
                    </van-card>
                </div>
                <van-cell>
                    <span slot="title">小计 <span
                            style="font-size: large">￥{{orderDetail.transaction.order_price}}</span></span>
                </van-cell>
                <div style="padding-top: 20px">
                    <van-cell-group>
                        <van-cell title="下单时间"
                                  :value="Date(orderDetail.transaction.order_time).substring(0, 24)"></van-cell>
                        <van-cell v-if='orderDetail.transaction.order_status === 5' title="确认时间"
                                  :value="orderDetail.transaction.order_confirm_time"></van-cell>
                        <van-cell title="订单备注" :value="orderDetail.order_script"></van-cell>
                        <van-cell title="餐具份数" :value="orderDetail.table_ware_num"></van-cell>
                        <van-cell title="发票信息" value="未选择"></van-cell>
                    </van-cell-group>
                </div>
                <div v-if="orderDetail.transaction.order_status === 1" style="padding-top: 20px;">
                    <van-button type="default" size="large" round style="background-color: #fed76f"
                                @click="pay">支付
                    </van-button>
                </div>
                <div v-if="orderDetail.transaction.order_status === 1" style="padding-top: 20px;">
                    <van-button type="default" size="large" round style="background-color: lightgray"
                                @click="deleteOrder">取消订单
                    </van-button>
                </div>
                <div v-if="orderDetail.transaction.order_status === 4" style="padding-top: 20px;">
                    <van-button type="default" size="large" round style="background-color: #fed76f"
                                @click="changeOrderState">确认订单
                    </van-button>
                </div>
            </van-col>
        </van-row>
    </div>
</template>

<script>
    export default {
        name: "OrderDetail",
        data() {
            return {
                orderId: Number,
                orderDetail: Array,
                noUse: null,
                userDishNum:[],
                dishList:[]
            }
        },
        created() {
            if(localStorage.getItem('orderDetail') === null) {
                this.orderDetail = this.$route.params.data;
                localStorage.setItem('orderDetail',JSON.stringify(this.orderDetail));
            }else{
                this.orderDetail = JSON.parse(localStorage.getItem('orderDetail'));
            }
        },
        destroyed(){
            localStorage.removeItem('orderDetail');
        },
        methods: {
            goBack() {
                this.$router.push({name: 'order'});
            },
            orderStatus(index) {
                if (index === 0) {
                    return '订单已取消';
                } else if (index === 1) {
                    return '已下单';
                } else if (index === 2) {
                    return '订单已支付';
                } else if (index === 3) {
                    return '订单处理中';
                } else if (index === 4) {
                    return '订单已完成';
                } else if (index === 5) {
                    return '订单已确认';
                }
            },
            changeOrderState() {
                this.$axios({
                    method: 'put',
                    url: '/orders/' + this.orderDetail.order_id + '/process/',
                    headers: {
                        "Authorization": "Bearer " + localStorage.getItem('user_token')
                    },
                    data: {
                        order: this.orderDetail.order_id,
                        order_status: 5
                    }
                })
                    .then(response => {
                        this.noUse = response;
                        this.$dialog.alert({
                            message:'订单确认成功！'
                        });
                        this.$router.push({name:'order'});
                    })
                    .catch(error => {
                        this.noUse = error;
                        this.$dialog.alert({
                            message: '出现错误'
                        })
                    })
            },
            deleteOrder(){
                this.$axios({
                    method: 'put',
                    url: '/orders/' + this.orderDetail.order_id + '/process/',
                    headers: {
                        "Authorization": "Bearer " + localStorage.getItem('user_token')
                    },
                    data: {
                        order: this.orderDetail.order_id,
                        order_status: 0
                    }
                })
                    .then(response => {
                        this.noUse = response;
                        this.$dialog.alert({
                            message:'订单取消成功！'
                        });
                        this.$router.push({name:'order'});
                    })
                    .catch(error => {
                        if(error.response.status === 400) {
                            this.$dialog.alert({
                                message: '此订单正在处理中，用户无法取消，如果想要取消，请联系商家！'
                            });
                            this.$router.push({name:'order'});
                        } else if(error.response.status === 401) {
                            this.$dialog.alert({
                                message: '登录已失效，请重新登录'
                            });
                            localStorage.setItem('user_name','');
                            this.$router.push({name:'login'});
                        }
                    })
            },
            pay(){
                localStorage.setItem('payUrl','http://120.24.91.195:8000/orders/pay?id='+this.orderDetail.order_id);
                this.$router.push({name:'confirmPay'});
            }
        }
    }
</script>

<style scoped>
    .iconStyle {
        padding-top: 20px;
        padding-bottom: 50px;
    }
</style>