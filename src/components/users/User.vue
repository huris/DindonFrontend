<template>
    <div>
        <van-row type="flex" justify="center" style="padding-top: 46px">
            <van-col span="18">
                <van-row>
                    <span style="font-family: 'Microsoft YaHei';font-size: 2rem">{{userName}}</span>
                </van-row>
                <van-row>
                    <span v-if="userName !== '未登录'" style="color: lightgrey;">再忙，也要记得吃饭哟~</span>
                </van-row>
            </van-col>
            <van-col span="4">
                <div style="height: 100%; width: 100%">
                    <img src="../../assets/logo.png" style="height: 100%;width: 100%;"/>
                </div>
            </van-col>
        </van-row>
        <div style="padding-top: 20px">
            <van-cell-group>
                <van-cell title="设置" icon="setting-o" is-link
                          :arrow-direction="this.direction[0] === 0 ? 'up' : 'down'" @click="changeDir(0)"/>
                <van-cell-group v-show="direction[0] === 1">
                    <van-cell title="修改密码" is-link @click="goToModify"></van-cell>
                    <van-cell title="注销" is-link @click="logout"></van-cell>
                </van-cell-group>
                <van-cell title="关于我们" icon="like-o" is-link
                          :arrow-direction="this.direction[1] === 0 ? 'up' : 'down'" @click="changeDir(1)"/>
                <van-cell v-show="direction[1] === 1">Supported by Vue.js and Vant.</van-cell>
            </van-cell-group>
        </div>
        <TabBar></TabBar>
    </div>
</template>

<script>
    import TabBar from "../base/TabBar";

    export default {
        name: "user",
        components: {TabBar},
        data() {
            return {
                direction: [],
                userName:String
            }
        },
        created() {
            for (let i = 0; i < 2; i++)
                this.direction[i] = 0;
            this.userName = localStorage.getItem('user_name');
            if(this.userName === '')
                this.userName = '未登录';
        },
        methods: {
            changeDir(index) {
                if (this.direction[index] === 0)
                    this.$set(this.direction, index, 1);
                else
                    this.$set(this.direction, index, 0);
            },
            goToModify() {
                this.$router.push({name:'modifypassword'});
            },
            logout() {
                localStorage.setItem("user_token",'');
                localStorage.setItem('user_name','');
                this.$router.push({name: 'login'});
            }
        }
    }
</script>

<style scoped>
    html {
        font-size: calc(100vw);
    }
</style>