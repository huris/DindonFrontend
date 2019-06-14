<template>
    <div>
        <div>
            <van-nav-bar
                    left-arrow
                    border
                    @click-left="onClickLeft"
            />
            <div style="padding-top: 3rem">
                <van-row type="flex" justify="center">
                    <span style="font-family: 'Microsoft YaHei';font-size: xx-large">欢迎预定！</span>
                </van-row>
            </div>
            <div style="padding-top: 4rem; font-family: 'Microsoft YaHei'">
                <van-row type="flex" justify="center">
                    <van-col span="22">
                        <div>
                            <van-cell-group>
                                <van-field
                                        v-model="tableId"
                                        label="餐桌编号"
                                        size="large"
                                        disabled
                                ></van-field>
                                <van-field
                                        v-model="bookDate"
                                        label="预定时间"
                                        size="large"
                                        @click="chooseTime"
                                ></van-field>
                                <van-popup v-model="show" position="bottom" @click-overlay="closePopUp">
                                    <van-datetime-picker
                                            v-model="currentDate"
                                            type="date"
                                            :min-date="minDate"
                                            :max-date="maxDate"
                                            @cancel="closePopUp"
                                            @confirm="confirmChoose"
                                            @change="getNowChoose"
                                    />
                                </van-popup>
                            </van-cell-group>
                        </div>
                        <div>
                            <van-radio-group :value="radio">
                                <van-cell-group>
                                    <van-cell clickable  @click="changeRadio('1')">
                                        <van-radio name="1" @click="changeRadio('1')"/>
                                        <span slot="title">上午</span>
                                    </van-cell>
                                    <van-cell clickable  @click="changeRadio('2')">
                                        <van-radio name="2" @click="changeRadio('2')"/>
                                        <span slot="title">中午</span>
                                    </van-cell>
                                    <van-cell clickable  @click="changeRadio('3')">
                                        <van-radio name="3" @click="changeRadio('3')"/>
                                        <span slot="title">下午</span>
                                    </van-cell>
                                    <van-cell clickable  @click="changeRadio('4')">
                                        <van-radio name="4" @click="changeRadio('4')"/>
                                        <span slot="title">晚上</span>
                                    </van-cell>
                                </van-cell-group>
                            </van-radio-group>
                        </div>
                    </van-col>
                </van-row>
                <van-row type="flex" justify="center" style="padding-top: 50px">
                    <van-col span="22">
                        <van-button style="background-color: green; color: white" round size="large"
                                    @click="bookTable">确认
                        </van-button>
                    </van-col>
                </van-row>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "SubmmitBook",
        data() {
            return {
                tableId: String,
                show: false,
                currentDate: Date,
                minDate: Date,
                maxDate: Date,
                bookDate: String,
                nowChoose: '',
                radio: String,
                isChange: false,
                noUse: null
            }
        },
        created() {
            this.currentDate = new Date();
            this.minDate = this.currentDate;
            this.maxDate = new Date(2021, 1, 1);
            if (localStorage.getItem('orderTableId') === null) {
                this.tableId = this.$route.params.id;
                localStorage.setItem('orderTableId', this.tableId);
            } else
                this.tableId = localStorage.getItem('orderTableId');
            if (localStorage.getItem('bookDate') === null) {
                this.bookDate = '';
                localStorage.setItem('bookDate', '');
            } else
                this.bookDate = localStorage.getItem('bookDate');
            if (localStorage.getItem('radio') === null) {
                this.radio = '1';
                localStorage.setItem('radio', '1');
            } else
                this.radio = localStorage.getItem('radio');
        },
        destroyed(){
            this.clearInfo();
        },
        methods: {
            onClickLeft() {
                this.$router.push({name: 'choosetable'});
            },
            bookTable() {
                if (this.bookDate === '') {
                    this.$dialog.alert({
                        message: '请先选择日期'
                    })
                } else if (this.tableId === '') {
                    this.$dialog.alert({
                        message: '请先填写餐桌号'
                    })
                } else {
                    this.$axios({
                        method: 'post',
                        url: '/tables/books',
                        headers: {
                            "Authorization": "Bearer " + localStorage.getItem('user_token')
                        },
                        data: {
                            "table": this.tableId,
                            "book_date": this.bookDate,
                            "book_time": this.radio
                        }
                    })
                        .then(data => {
                            this.noUse = data;
                            this.clearInfo();
                            this.$router.push({name: 'table'});
                        })
                        .catch(error => {
                            this.noUse = error;
                            if(error.response.status === 401) {
                                this.$dialog.alert({
                                    message: '登录已失效，请重新登录！'
                                });
                                this.$router.push({name:'login'});
                                localStorage.setItem('user_name','');
                            } else if(error.response.status === 400) {
                                this.$dialog.alert({
                                    message: '该餐桌此时间段已被预订，请另选时间！'
                                });
                            }
                        })
                }
            },
            chooseTime() {
                this.show = true;
                this.nowChoose = '';
            },
            closePopUp() {
                this.show = false;
            },
            getNowChoose(picker) {
                this.nowChoose = '';
                this.isChange = true;
                let value = picker.getValues();
                this.nowChoose += value[0];
                for (let i = 1; i < value.length; i++)
                    this.nowChoose = this.nowChoose + '-' + value[i];
            },
            confirmChoose() {
                if (this.isChange)
                    this.bookDate = this.nowChoose;
                else {
                    let month = this.currentDate.getMonth() + 1;
                    let day = this.currentDate.getDate();
                    if (month < 10)
                        month = '0' + month;
                    if (day < 10)
                        day = '0' + day;
                    this.bookDate = this.currentDate.getFullYear() + '-' + month + '-' + day;
                }
                localStorage.setItem('bookDate', this.bookDate);
                this.show = false;
            },
            changeRadio(index) {
                this.radio = index;
                localStorage.setItem('radio', index);
            },
            clearInfo(){
                localStorage.removeItem('radio');
                localStorage.removeItem('bookDate');
                localStorage.removeItem('orderTableId');
            }
        }
    }
</script>

<style scoped>

</style>