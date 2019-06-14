<template>
    <div>
        <van-nav-bar
                right-text="帮助"
                left-arrow
                border
                @click-left="onClickLeft"
        />
        <div style="padding-top: 3rem">
            <van-row type="flex" justify="center">
                <span style="font-family: 'Microsoft YaHei',serif;font-size: xx-large">修改密码</span>
            </van-row>
        </div>
        <div style="padding-top: 4rem; font-family: 'Microsoft YaHei',serif">
            <van-row type="flex" justify="center">
                <van-col span="22">
                    <van-cell-group>
                        <van-field
                                v-model="oldPassword"
                                label="原密码"
                                placeholder="请输入原密码"
                                type="password"
                                size="large"
                                @input="passwordblur"
                        ></van-field>
                        <van-field
                                v-model="newPassword"
                                label="新密码"
                                placeholder="请输入新密码"
                                type="password"
                                size="large"
                                @input="newPasswordblur"
                        ></van-field>
                        <van-field
                                v-model="checkCode"
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
                            <van-button v-else slot="button" size="small" disabled style="background-color: lightgrey;">
                                重新发送({{checkcodeTime}}s)
                            </van-button>
                        </van-field>
                    </van-cell-group>

                    <div style="padding-top: 2rem">
                        <van-button style="background-color: green; color: white" round size="large"
                                    @click="modifyPassword">确认
                        </van-button>
                    </div>
                </van-col>
            </van-row>
        </div>
    </div>
</template>

<script>
    export default {
        name: "ModifyPassword",
        data() {
            return {
                isCheckCode: true,
                checkcodeTime: Number,
                noUse: null,
                oldPassword:String,
                newPassword:String,
                checkCode:String
            }
        },
        created() {
            if(localStorage.getItem('oldPassword')===null){
                this.oldPassword = '';
                localStorage.setItem('oldPassword','');
            } else {
                this.oldPassword  =localStorage.getItem('oldPassword');
            }
            if(localStorage.getItem('newPassword')===null){
                this.newPassword = '';
                localStorage.setItem('newPassword','');
            } else {
                this.newPassword  =localStorage.getItem('newPassword');
            }
            if(localStorage.getItem('checkCode')===null){
                this.checkCode = '';
                localStorage.setItem('checkCode','');
            } else {
                this.checkCode  =localStorage.getItem('checkCode');
            }
        },
        destroyed(){
            localStorage.removeItem('oldPassword');
            localStorage.removeItem('newPassword');
            localStorage.removeItem('checkCode');
        },
        methods: {
            passwordblur(value) {
                this.oldPassword = value;
                localStorage.setItem('oldPassword',this.oldPassword);
            },
            newPasswordblur(value) {
                this.newPassword = value;
                localStorage.setItem('newPassword',this.newPassword);
            },
            checkCodeblur(value) {
                this.checkCode = value;
                localStorage.setItem('checkCode',this.checkCode);
            },
            sendCheckCode() {
                if (this.oldPassword === '') {
                    this.$dialog.alert({
                        message: '原密码为空！'
                    })
                } else if (this.newPassword === '') {
                    this.$dialog.alert({
                        message: '新密码为空！'
                    })
                } else if (this.newPassword.length < 6) {
                    this.$dialog.alert({
                        message: '新密码长度不能少于6位！'
                    })
                } else {
                    this.$axios.post('/users/code/', {
                        "phone_number": localStorage.getItem('user_name'),
                        "purpose": 2
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
                }
            },
            onClickLeft() {
                this.$router.go(-1);
            },
            modifyPassword() {
                this.$axios({
                    url: '/users/changepassword/' + localStorage.getItem('uid') + '/',
                    method: "put",
                    headers: {
                        "Authorization": "Bearer " + localStorage.getItem('user_token')
                    },
                    data: {
                        "old_password": this.oldPassword,
                        "new_password": this.newPassword,
                        "code": this.checkCode
                    }
                })
                    .then(response => {
                        this.noUse = response;
                        this.$dialog.alert({
                            message: '修改成功！'
                        });
                        localStorage.removeItem('oldPassword');
                        localStorage.removeItem('newPassword');
                        localStorage.removeItem('checkCode');
                        this.$router.push({name: 'user'});
                    })
                    .catch(error => {
                        if(error.response.status === 400) {
                            this.$dialog.alert({
                                message: '原密码错误！'
                            });
                        }
                    })
            }
        }
    }
</script>

<style scoped>

</style>