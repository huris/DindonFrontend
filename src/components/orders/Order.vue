<template>
    <div>
        <div v-if="isHaveOrder === 0" style="padding-top: 40px">
            <van-row type="flex" justify="center">
                <span style="font-family: 'Microsoft YaHei',serif; color: lightgrey">您还没有相关订单哟</span>
            </van-row>
            <van-row type="flex" justify="center">
                <a style="text-decoration: underline; font-family: 'Microsoft YaHei',serif; color: lightgrey"
                   @click="goToDish">点此点单</a>
            </van-row>
        </div>
        <div v-else style="padding-top: 50px">
            <van-row type="flex" justify="center">
                <van-col span="23">
                    <span style="font-family: 'Microsoft YaHei',serif; font-size: 1.5rem">历史订单</span>
                </van-col>
            </van-row>
            <van-row type="flex" justify="center" style="padding-top: 20px">
                <van-col span="23">
                    <van-panel v-for="(item,index) in orderList" :key="index" style="padding-bottom: 10px">
                        <div slot="header">
                            <van-row type="flex">
                                <van-col span="3">
                                    <div style="height: 100%;width: 100%">
                                        <img src="../../assets/logo.png" style="height: 100%; width: 100%;" alt=""/>
                                    </div>
                                </van-col>
                                <van-col span="15" style="padding-left: 10px">
                                    <van-row>
                                        <span style="font-weight: bold">叮咚叮咚(DinDonDinDon)</span>
                                    </van-row>
                                    <van-row style="padding-top: 5px">
                                        <span v-if="item.transaction.order_status === 0 || item.transaction.order_status === 1" style="color: grey; font-size: 0.8rem">{{item.transaction.order_time.substring(0, 10)+' '+item.transaction.order_time.substring(11, 19)}}</span>
                                        <span v-if="item.transaction.order_status === 2" style="color: grey; font-size: 0.8rem">{{item.transaction.order_pay_time.substring(0, 10)+' '+item.transaction.order_pay_time.substring(11, 19)}}</span>
                                        <span v-if="item.transaction.order_status === 3" style="color: grey; font-size: 0.8rem">{{item.transaction.order_process_time.substring(0, 10)+' '+item.transaction.order_process_time.substring(11, 19)}}</span>
                                        <span v-if="item.transaction.order_status === 4" style="color: grey; font-size: 0.8rem">{{item.transaction.order_finish_time.substring(0, 10)+' '+item.transaction.order_finish_time.substring(11, 19)}}</span>
                                        <span v-if="item.transaction.order_status === 5" style="color: grey; font-size: 0.8rem">{{item.transaction.order_confirm_time.substring(0, 10)+' '+item.transaction.order_confirm_time.substring(11, 19)}}</span>
                                    </van-row>
                                </van-col>
                                <van-col span="5">
                                    <van-row type="flex" justify="end">
                                        <span v-if="item.transaction.order_status === 0"
                                              style="font-size: 0.8rem">订单已取消</span>
                                        <span v-if="item.transaction.order_status === 1"
                                              style="font-size: 0.8rem">已下单</span>
                                        <span v-if="item.transaction.order_status === 2"
                                              style="font-size: 0.8rem">订单已支付</span>
                                        <span v-if="item.transaction.order_status === 3"
                                              style="font-size: 0.8rem">订单处理中</span>
                                        <span v-if="item.transaction.order_status === 4"
                                              style="font-size: 0.8rem">订单已完成</span>
                                        <span v-if="item.transaction.order_status === 5"
                                              style="font-size: 0.8rem">订单已确认</span>
                                    </van-row>
                                    <van-row></van-row>
                                </van-col>
                            </van-row>
                        </div>
                        <van-row type="flex" justify="end" style="padding-top: 5px">
                            <van-col span="15">
                                <span style="padding-top: 20px; font-size: 0.8rem">共{{item.order_detail.length}}件商品</span>
                            </van-col>
                            <van-col span="5">
                                <span style="padding-top: 20px; font-size: 0.8rem">￥{{item.transaction.order_price}}</span>
                            </van-col>
                        </van-row>
                        <div slot="footer">
                            <van-row type="flex" justify="end">
                                <van-button size="small" @click="showOrderDetail(item)">查看详情</van-button>
                            </van-row>
                        </div>
                    </van-panel>
                </van-col>
            </van-row>
        </div>
        <div style="padding-bottom: 50px"></div>
        <TabBar></TabBar>
    </div>
</template>

<script>
    import TabBar from "../base/TabBar";

    export default {
        name: "order",
        components: {
            TabBar
        },
        data() {
            return {
                isHaveOrder: 0,
                orderList: Array,
                noUse: null
            }
        },
        created() {
            this.$axios({
                method: 'get',
                url: '/orders/?page_size=100',
                headers: {
                    "Authorization": "Bearer " + localStorage.getItem('user_token')
                }
            })
                .then(response => {
                    this.orderList = response.data.results.reverse();
                    this.isHaveOrder = response.data.count;
                })
                .catch(error => {
                    if (error.response.status === 401) {
                        this.$dialog.alert({
                            message: '登录已失效，请重新登录'
                        });
                        localStorage.setItem('user_name', '');
                        this.$router.push({name: 'login'});
                    }
                })
        },
        methods: {
            goToDish() {
                this.$router.push({name: 'dish'});
            },
            showOrderDetail(index) {
                this.$router.push({name: 'orderdetail', params: {data: index}});
            }
        }
    }
</script>

<style scoped>
    html {
        font-size: calc(100vw);
    }
</style>