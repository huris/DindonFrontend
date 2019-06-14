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
                    <span style="font-family: 'Microsoft YaHei';font-size: xx-large">修改预定时间</span>
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
                            <van-col span="10">
                                <van-radio-group v-model="radio">
                                    <van-cell-group>
                                        <van-cell title="上午" clickable border @click="changeRadio('1')">
                                            <van-radio name="1"/>
                                        </van-cell>
                                        <van-cell title="中午" clickable border @click="changeRadio('2')">
                                            <van-radio name="2"/>
                                        </van-cell>
                                        <van-cell title="下午" clickable border @click="changeRadio('3')">
                                            <van-radio name="3"/>
                                        </van-cell>
                                        <van-cell title="晚上" clickable border @click="changeRadio('4')">
                                            <van-radio name="4"/>
                                        </van-cell>
                                    </van-cell-group>
                                </van-radio-group>
                            </van-col>
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
        name: "changeBook",
        data() {
            return {
                tableId: Number,
                recordId: Number,
                show: false,
                currentDate: '',
                minDate: '',
                maxDate: '',
                bookDate: '',
                nowChoose: '',
                radio: '1',
                isChange: false,
                host: this.$store.state.host,
                noUse: null
            }
        },
        created() {
            this.currentDate = new Date();
            this.minDate = this.currentDate;
            this.maxDate = new Date(2021, 1, 1);
            this.tableId = this.$route.params.id;
            this.recordId = this.$route.params.recordId;
        },
        methods: {
            onClickLeft() {
                this.$router.push({name: 'table'});
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
                        method: 'put',
                        url: '/tables/books/' + this.recordId + '/update',
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
                            this.$router.push({name: 'table'});
                        })
                        .catch(error => {
                            this.noUse = error;
                            this.$dialog.alert({
                                message: '出现错误，请重试！'
                            })
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
                this.show = false;
            },
            changeRadio(index) {
                this.radio = index;
            },
            getTableId(value) {
                this.tableId = value;
            }
        }
    }
</script>

<style scoped>

</style>