<template>
    <div>
        <van-row class="swipeSection">
            <van-swipe :autoplay="6000" indicator-color="yellow">
                <van-swipe-item v-for="(image, index) in imges" :key="index">
                    <img class="swipeImg" :src="image.announcement_picture" alt="dish_picture">
                </van-swipe-item>
            </van-swipe>
        </van-row>
    </div>
</template>
<script>
    export default {
        name: "dishSwipe",
        data() {
            return {
                imges: Array,
                noUse: null
            }
        },
        created() {
            this.$axios({
                url: '/announcements/',
                method: 'get'
            })
                .then(response => {
                    this.imges = response.data.results;
                })
                .catch(error => {
                    if(error.response.status === 503) {
                        this.$dialog.alert({
                            message: '当前服务不可得到！'
                        });
                    } else if (error.response.status === 502){
                        this.$dialog.alert({
                            message: '当前网络不佳！'
                        });
                    }
                })
        }
    };
</script>

<style scoped>
    .swipeSection {
        height: 40%;
    }

    .swipeImg {
        width: 100%;
        height: 100%;
    }
</style>