<template>
    <div class="me">
        <div class="top">
            <!-- <img src="../../../assets/images/pic.jpg" class="avatar"> -->
            <el-avatar :size="60" style="user-select: none; font-size: 16px;">{{firstName}}</el-avatar>
            <div class="character">
                <h3>{{user.username}}</h3>
                <p>余额:{{user.balance}}</p>
            </div>
        </div>
        <div class="content">
            <van-icon name="add" style="margin-right: 6px" />
            <span style="flex: 1" @click="release">我的发布</span>
            <van-icon name="arrow" />
        </div>
        <div class="content">
            <van-icon name="thumb-circle" style="margin-right: 6px" />
            <span style="flex: 1" @click="help">我的帮助</span>
            <van-icon name="arrow" />
        </div>
        <div class="content">
            <van-icon name="gold-coin" style="margin-right: 6px" />
            <span style="flex: 1" @click="show = true">充值余额</span>
            <van-icon name="arrow" />
        </div>
        <div class="content">
            <van-icon name="warning" style="margin-right: 6px" />
            <span style="flex: 1" @click="$msg('敬请期待','error')">关于我们</span>
            <van-icon name="arrow" />
        </div>
        <div class="content">
            <van-icon name="setting" style="margin-right: 6px" />
            <span style="flex: 1" @click="$msg('敬请期待','error')">设置中心</span>
            <van-icon name="arrow" />
        </div>

        <van-action-sheet v-model="show" title="充值金额">
            <div>
                <van-field v-model="balance" type="number" left-icon="balance-o" />
                <el-button type="primary" style="float: right;margin: 20px" @click="recharge(user.id, user.studentId)">提交</el-button>
            </div>
        </van-action-sheet>
    </div>
</template>

<script>
    import {mapState,mapMutations, textAvatar} from 'vuex';
    export default {
        data(){
          return{
              show:false,
              balance:'',
              firstName:''
          }
        },
        name: "Me",
        computed:{
            ...mapState('user', ['user'])
        },
        created() {
            console.log(this.user.username)
            let arr = this.user.username.split(' ');
            for (var i in arr) {
                this.firstName += arr[i].substr(0,1);
            }
            this.firstName = this.firstName.toLocaleUpperCase();
            console.log('firstName->' + this.firstName);
        },
        methods:{
            ...mapMutations(['setActive']),
            ...mapMutations('user', ['setUser']),
            release(){
                this.setActive(1)
                sessionStorage.setItem("active",1)
                this.$router.push({name:'mTask'})
            },
            help(){
                this.setActive(1)
                sessionStorage.setItem("active",1)
                this.$router.push({name:'mTask',params:{index:1}})

            },
            recharge(id, studentId){
                this.$put('/user/rollIn', {studentId: studentId, balance: this.balance})
                .then((res) => {
                    this.$get("/user/" + id)
                    .then((rs) => {
                        sessionStorage.setItem("user", JSON.stringify(rs.data.user))
                        this.setUser(JSON.parse(sessionStorage.getItem("user")))
                    })
                    this.$msg( '充值成功', 'success')
                    this.balance = ''
                    this.show = false
                })
            }

        }
    }
</script>

<style scoped lang="less">
    .top {
        height: 30vh;
        color: white;
        background: rgb(138, 151, 246);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;

        .avatar {
            box-sizing: border-box;
            width: 22vw;
            /*animation: mymove .5s linear infinite;*/
            border-radius: 50%;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        .character {
            margin: 10px 0;
            text-align: center;

            h3 {
                font-weight: normal;
                margin-bottom: 5px;
            }

            p {
                font-size: 14px;
            }
        }

    }

    .content{
        padding: 10px 16px;
        font-size: 14px;
        line-height: 24px;
        background-color: #f4f5f8;
        border-bottom: 1px solid #ebedf0;
        display: flex;
        align-items: center;
    }

    @keyframes mymove {
        0% {
            padding: 0
        }
        50% {
            padding: 5px
        }
        100% {
            padding: 0
        }
    }

    .van-cell::after{
        border-bottom: 1px solid #cacaca !important;
    }
</style>