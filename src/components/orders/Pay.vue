<template>
    <div class="note">
        <div>
            <div style="padding-top: 3rem;">
                <van-row type="flex" justify="center">
                    <span style="font-family: 'Microsoft YaHei',serif;font-size: xx-large">确认支付</span>
                </van-row>
            </div>
            <div style="padding-top: 4rem; font-family: 'Microsoft YaHei',serif">
                <van-row type="flex" justify="center">
                    <van-col span="22">
                        <div style="padding-top: 2rem; opacity: 0.7">
                            <van-button style="background-color: green; color: white" round size="large" @click="confirmPay">确认
                            </van-button>
                        </div>
                    </van-col>
                </van-row>
                <van-row type="flex" justify="center">
                    <van-col span="22">
                        <div style="padding-top: 2rem; opacity: 0.7">
                            <div style="padding-top: 2rem; opacity: 0.7">
                                <van-button style="background-color:lightgray; color: white" round size="large" @click="concelPay">取消
                                </van-button>
                            </div>
                        </div>
                    </van-col>
                </van-row>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Pay",
        data() {
            return {
                payUrl: String,
                noUse: null
            }
        },
        created() {
            this.payUrl = localStorage.getItem('payUrl');
        },
        destroyed(){
          localStorage.removeItem('payUrl');
        },
        methods: {
            confirmPay(){
                this.$axios({
                    url: this.payUrl,
                    method: 'get',
                    headers: {
                        Authorization: "Bearer " + localStorage.getItem('user_token')
                    }
                })
                    .then(data => {
                        this.noUse = data;
                        this.$dialog.alert({
                            message: '支付成功！'
                        });
                        this.$router.push({name: 'order'});
                    })
                    .catch(error => {
                        this.noUse = error;
                        this.$dialog.alert({
                            message: '出现错误！'
                        });
                    })
            },
            concelPay(){
                this.$dialog.confirm({
                    message:'确定取消支付？'
                })
                    .then(response => {
                        this.noUse = response;
                        this.$router.push({name:'order'});
                    })
                    .catch(error => {
                        this.noUse = error;
                    })
            }
        }
    }
</script>

<style scoped>

</style>