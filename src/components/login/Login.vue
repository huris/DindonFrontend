<template>
    <div class="note">
        <div>
            <van-nav-bar
                    right-text="帮助"
                    left-arrow
                    border
                    @click-left="onClickLeft"
                    @click-right="onClickRight"
            />
            <div style="padding-top: 3rem;">
                <van-row type="flex" justify="center">
                    <span style="font-family: 'Microsoft YaHei',serif;font-size: xx-large">欢迎登录！</span>
                </van-row>
            </div>
            <div style="padding-top: 4rem; font-family: 'Microsoft YaHei',serif">
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
                                    v-if="!isPasswordLogin"
                                    center
                                    clearable
                                    label="短信验证码"
                                    placeholder="请输入短信验证码"
                                    size="large"
                                    @input="checkCodeblur"
                            >
                                <van-button v-if="isCheckCode" slot="button" size="small" type="primary"
                                            @click="sendCheckCode">发送验证码
                                </van-button>
                                <van-button v-else slot="button" size="small" disabled
                                            style="background-color: lightgrey;">
                                    重新发送({{checkcodeTime}}s)
                                </van-button>
                            </van-field>
                            <van-field
                                    v-else
                                    v-model="password"
                                    label="密码"
                                    placeholder="请输入密码"
                                    type="password"
                                    size="large"
                                    @input="passwordblur"
                            ></van-field>
                        </van-cell-group>
                        <van-row type="flex" justify="space-around">
                            <van-col span="15">
                                <a v-if="!isPasswordLogin"
                                   class="fontStyle"
                                   @click="changeLoginWay">密码登录请点这:)</a>
                                <a v-else
                                   class="fontStyle"
                                   @click="changeLoginWay">验证码登录请点这:)</a>
                            </van-col>
                            <van-col span="7">
                                <a class="fontStyle" @click="goToRegister">新用户请点这:)</a>
                            </van-col>
                        </van-row>
                        <div style="padding-top: 2rem; opacity: 0.7">
                            <van-button style="background-color: green; color: white" round size="large" @click="login">
                                登录
                            </van-button>
                        </div>
                    </van-col>
                </van-row>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data() {
            return {
                isLegal: Boolean,
                isCheckCode: Boolean,
                checkcodeTime: Number,
                timer: null,
                isPasswordLogin: Boolean,
                noUse: null,
                phoneNumber: String,
                password: String,
                checkCode: String
            }
        },
        created() {
            this.isLegal = false;
            this.isPasswordLogin = false;
            this.checkcodeTime = 60;
            this.isCheckCode = true;
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
            sendCheckCode() {
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
                        "purpose": 1
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
                            if (error.response.status === 400) {
                                this.$dialog.alert({
                                    message: '该手机号未注册，请先注册！'
                                });
                                this.$router.push({name: 'register'});
                            }
                        })
                }
            },
            login() {
                if (this.phoneNumber === '') {
                    this.dialogMessage = '手机号不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (!this.isLegal) {
                    this.dialogMessage = '手机号格式不正确';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (!this.isPasswordLogin && this.checkCode === '') {
                    this.dialogMessage = '验证码不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else if (this.isPasswordLogin && this.password === '') {
                    this.dialogMessage = '密码不能为空';
                    this.$dialog.alert({
                        message: this.dialogMessage
                    })
                } else {
                    if (this.isPasswordLogin) {
                        this.$axios.post('/users/login/password/', {
                            phone_number: this.phoneNumber,
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
                            .catch(response => {
                                if (response.response.status === 401) {
                                    this.$dialog.alert({
                                        message: '密码错误！'
                                    })
                                }
                            })
                    } else {
                        this.$axios.post('/users/login/code/', {
                            phone_number: this.phoneNumber,
                            code: this.checkCode
                        })
                            .then(data => {
                                localStorage.setItem("user_name", data.data.username);
                                localStorage.setItem("user_token", data.data.access);
                                localStorage.setItem("refresh_token", data.data.refresh);
                                localStorage.setItem("uid", data.data.uid);
                                this.clearInfo();
                                this.$router.push({name: 'announcement'});
                            })
                            .catch(error => {
                                if (error.response.status === 400) {
                                    this.$dialog.alert({
                                        message: '验证码码错误！'
                                    })
                                }
                            })
                    }
                }
            },
            goToRegister() {
                this.clearInfo();
                this.$router.push({name: 'register'});
            },
            changeLoginWay() {
                this.isPasswordLogin = !this.isPasswordLogin;
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

    .fontStyle {
        color: lightgray;
        padding-top: 5px;
        text-decoration: underline;
        font-family: 'Microsoft YaHei', serif;
    }
</style>