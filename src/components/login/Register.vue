<template>
    <div>
        <van-nav-bar
                right-text="帮助"
                left-arrow
                border
                @click-left="onClickLeft"
                @click-right="onClickRight"
        />
        <div style="padding-top: 3rem">
            <van-row type="flex" justify="center">
                <span style="font-family: 'Microsoft YaHei',serif;font-size: xx-large">欢迎注册！</span>
            </van-row>
        </div>
        <div style="padding-top: 4rem">
            <van-row type="flex" justify="center">
                <van-col span="22">
                    <van-cell-group>
                        <van-field
                                v-model="phoneNumber"
                                label="手机号"
                                placeholder="请输入手机号"
                                size="large"
                                @input="phoneNumberblur"
                        ></van-field>
                        <van-field
                                v-model="checkCode"
                                center
                                label="短信验证码"
                                placeholder="请输入短信验证码"
                                size="large"
                                @input="checkCodeblur"
                        >
                            <van-button v-if="isCheckCode" slot="button" size="small" type="primary"
                                        @click="cendCheckCode">发送验证码
                            </van-button>
                            <van-button v-else slot="button" size="small" disabled style="background-color: lightgrey;">
                                重新发送({{checkcodeTime}}s)
                            </van-button>
                        </van-field>
                        <van-field
                                label="密码"
                                v-model="password"
                                placeholder="请输入密码（不少于6位）"
                                size="large"
                                type="password"
                                @input="passwordblur"
                        ></van-field>
                    </van-cell-group>
                    <div style="padding-top: 2rem">
                        <van-button style="background-color: #fed76f;" round size="large" @click="register">注册
                        </van-button>
                    </div>
                </van-col>
            </van-row>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Register",
        data() {
            return {
                isLegal: Boolean,
                dialogMessage: String,
                isCheckCode: Boolean,
                checkcodeTime: Number,
                host: this.$store.state.host,
                noUse: null
            }
        },
        created() {
            this.dialogMessage = '';
            this.isLegal = false;
            this.isCheckCode = true;
            this.checkcodeTime = 60;
            if (localStorage.getItem('phoneNumber') === null) {
                this.phoneNumber = '';
                localStorage.setItem('phoneNumber', this.phoneNumber);
            } else {
                this.isLegal = false;
                this.phoneNumber = localStorage.getItem('phoneNumber');
                if (this.phoneNumber.length === 11) {
                    this.isLegal = true;
                    for (let i = 0; i < this.phoneNumber.length; i++) {
                        if (this.phoneNumber[i] < '0' || this.phoneNumber[i] > '9') {
                            this.isLegal = false;
                            break;
                        }
                    }
                }
            }
            if (localStorage.getItem('password') === null) {
                this.password = '';
                localStorage.setItem('password', this.password);
            } else {
                this.password = localStorage.getItem('password');
            }
            if (localStorage.getItem('checkCode') === null) {
                this.checkCode = '';
                localStorage.setItem('checkCode', this.checkCode);
            } else {
                this.checkCode = localStorage.getItem('checkCode');
            }
        },
        destroyed() {
            this.clearInfo();
        },
        methods: {
            onClickLeft() {
                this.$router.push({name: 'announcement'});
            },
            onClickRight() {

            },
            phoneNumberblur(value) {
                this.isLegal = false;
                this.phoneNumber = value;
                localStorage.setItem('phoneNumber', this.phoneNumber);
                if (this.phoneNumber.length === 11) {
                    this.isLegal = true;
                    for (let i = 0; i < this.phoneNumber.length; i++) {
                        if (this.phoneNumber[i] < '0' || this.phoneNumber[i] > '9') {
                            this.isLegal = false;
                            break;
                        }
                    }
                }
            },
            passwordblur(value) {
                this.password = value;
                localStorage.setItem('password', this.password);
            },
            checkCodeblur(value) {
                this.checkCode = value;
                localStorage.setItem('checkCode', this.checkCode);
            },
            register() {
                if (this.phoneNumber === '' && this.checkCode === '' && this.password === '') {
                    this.dialogMessage = '手机号、验证码和密码不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (this.phoneNumber === '') {
                    this.dialogMessage = '手机号不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (!this.isLegal) {
                    this.dialogMessage = '手机号格式不正确';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (this.checkCode === '') {
                    this.dialogMessage = '验证码不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (this.password === '') {
                    this.dialogMessage = '密码不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (this.password.length < 6) {
                    this.dialogMessage = '密码不能少于6位';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else {
                    this.$axios.post('/users/register/', {
                        phone_number: this.phoneNumber,
                        code: this.checkCode,
                        password: this.password
                    })
                        .then(data => {
                            localStorage.setItem("user_name", data.data.username);
                            localStorage.setItem("user_token", data.data.access);
                            localStorage.setItem("refresh_token", data.data.refresh);
                            localStorage.setItem("uid", data.data.uid);
                            this.clearInfo();
                            this.$router.push({name: 'announcement'});
                        })
                        .catch(function (error) {
                            if(error.response.status === 400){
                                this.$dialog.alert({
                                    message: '此手机号已注册,请直接登录！'
                                });
                                this.$router.push({name:'login'});
                            }
                        })
                }
            },
            cendCheckCode() {
                if (this.phoneNumber === '') {
                    this.$dialog.alert({
                        message: '未填写手机号！'
                    })
                } else if (!this.isLegal) {
                    this.$dialog.alert({
                        message: '手机号不合法！'
                    })
                } else {
                    this.$axios.post('/users/code/', {
                        "phone_number": this.phoneNumber,
                        "purpose": 0
                    })
                        .then(data => {
                            this.noUse = data;
                            const TIME_COUNT = 60;
                            if (!this.timer) {
                                this.checkcodeTime = TIME_COUNT;
                                this.isCheckCode = false;
                                this.timer = setInterval(() => {
                                    if (this.checkcodeTime > 0 && this.checkcodeTime <= TIME_COUNT) {
                                        this.checkcodeTime--;
                                    } else {
                                        this.isCheckCode = true;
                                        clearInterval(this.timer);
                                        this.timer = null;
                                    }
                                }, 1000)
                            }
                        })
                        .catch(error => {
                            if (error.response.data.non_field_errors[0] === '手机号格式错误') {
                                this.$dialog.alert({
                                    message: '手机号格式错误！'
                                })
                            } else if (error.response.data.non_field_errors[0] === '该手机号已经被注册') {
                                this.$dialog.alert({
                                    message: '此手机号已注册,请直接登录！'
                                })
                            }
                        })
                }
            },
            clearInfo() {
                localStorage.removeItem('phoneNumber');
                localStorage.removeItem('password');
                localStorage.removeItem('checkCode');
            }
        }
    }
</script>

<style scoped>
    html {
        font-size: calc(100vw);
    }
</style>