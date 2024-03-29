<template>
    <div class="content">
        <el-card class="box-card">
            <div slot="header" class="clearfix">
                <span>已发布任务</span>
            </div>
            <el-card class="box-card" v-for="item in tasks" style="margin-top: 20px">
                <div slot="header" class="clearfix"
                     style="display: flex; align-items: center; justify-content: space-between">
                        <span style="display: flex;align-items: center">
                            <el-tag :type="item.state == 0 ? 'danger':(item.state == 1 ? 'warning':'success')"
                                    style="margin-right: 5px">{{item.state == 0 ? '待解决':(item.state == 1 ? '服务中':'已完成')}}</el-tag>
                            {{item.taskTitle}}
                        </span>
                    <el-button style="float: right; padding: 3px 0" type="text" v-show="item.state != 0"
                               @click="receiver(item)">查看接受人信息
                    </el-button>
                    <template>
<!--                        <i class="el-icon-edit" style="cursor: pointer; color: #66b1ff" v-show="item.state == 0"/>-->
                        <el-popconfirm title="确定取消任务吗？" @confirm="cancel(item.id)" v-show="item.state == 0">
                            <el-button style="float: right; padding: 3px 0" type="text" slot="reference">取消任务
                            </el-button>
                        </el-popconfirm>
                    </template>
                </div>
                <el-steps :active="item.state + 1" finish-status="success">
                    <el-step title="发布成功" :description="item.createTime | formatDate"></el-step>
                    <el-step title="服务中" :description="item.orderTime ? transform(item.orderTime):'暂时没人服务'"></el-step>
                    <el-step title="完成时间" :description="item.endTime ? transform(item.endTime):''"></el-step>
                </el-steps>
                <el-collapse style="margin-top: 20px" v-model="activeNames">
                    <el-collapse-item title="任务内容" name="1">
                        <div>{{item.taskContext}}</div>
                    </el-collapse-item>
                    <el-collapse-item title="任务奖励" name="2">
                        <div><i class="el-icon-money" style="color: red;"> {{item.reward}}元</i></div>
                    </el-collapse-item>
                    <el-collapse-item title="发布时间" name="3">
                        <div>{{item.createTime | formatDate}}</div>
                    </el-collapse-item>
                </el-collapse>
                <el-button type="primary" style="float: right;margin:10px 0;" @click="completeTask(item.id)"
                           v-show="item.state==1">完成任务
                </el-button>
            </el-card>
            <div style="text-align: center" v-if="tasks.length == 0">
                <span><i class="el-icon-refresh-right"></i>暂无发布任务</span>
            </div>
        </el-card>
        <el-drawer
                title="接受人信息"
                :visible.sync="drawer"
                direction="rtl">
            <div class="content_drawer">
                <el-card class="box-card" v-if="recipientInformation != ''">
                    <el-collapse v-model="drawerNames">
                        <el-collapse-item title="姓名" name="1">
                            <div>{{recipientInformation.username}}</div>
                        </el-collapse-item>
                        <el-collapse-item title="电话" name="2">
                            <div>{{recipientInformation.phone}}</div>
                        </el-collapse-item>
                        <el-collapse-item title="学校" name="3">
                            <div>{{recipientInformation.school.name}}</div>
                        </el-collapse-item>
                        <el-collapse-item title="所在系" name="4">
                            <div>{{recipientInformation.dept.name}}</div>
                        </el-collapse-item>
                        <el-collapse-item title="所在班级" name="5">
                            <div>{{recipientInformation.aclass.name}}</div>
                        </el-collapse-item>
                    </el-collapse>
                </el-card>
            </div>
        </el-drawer>
    </div>
</template>

<script>
    import {mapState} from "vuex"
    import {formatDate} from '@/util/date';

    export default {
        name: "Published",
        data() {
            return {
                activeNames: ['1', '2', '3'],
                drawerNames: ['1', '2', '3', '4', '5'],
                tasks: [],
                drawer: false,
                recipientInformation: []
            };
        },
        computed: {
            ...mapState('user', ['user'])
        },
        created() {
            this.retrieveData()
        },
        methods: {
            retrieveData() {
                this.$get("/task/published", {id: this.user.id}).then(res => {
                    // console.log(res.data)
                    this.tasks = res.data.task
                })
            },
            receiver(val) {
                this.recipientInformation = val.accept;
                // console.log(this.recipientInformation)
                this.drawer = true
            },
            transform(time) {
                let date = new Date(time);
                return formatDate(date, 'yyyy-MM-dd hh:mm');
            },
            cancel(id) {
                this.$del("/task/" + id)
                    .then(res => {
                        this.retrieveData()
                        this.$notifyMsg('成功', res.data.msg, "success");
                    })
            },
            completeTask(id) {
                this.$msgbox({
                    title: '提示',
                    message: '确定接受人完成此任务了吗？',
                    showCancelButton: true,
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    beforeClose: ((action, instance, done) => {
                        if (action == 'confirm') {
                            // instance.confirmButtonLoading = true;
                            instance.confirmButtonText = '执行中...';
                            this.$put('task/' + id)
                                .then((res) => {
                                    done();
                                    instance.confirmButtonLoading = false;
                                    this.$msg(res.data.msg, "success");
                                    this.retrieveData()
                                })
                        } else {
                            done();
                        }
                    })
                }).catch(() => {
                })
            }
        },
        filters: {
            formatDate(time) {
                let date = new Date(time);
                return formatDate(date, 'yyyy-MM-dd hh:mm');
            }
        }
    }
</script>

<style scoped lang="less">
    .content {
        background: #FFf;
        margin: 0 15px;
        padding: 15px;
    }
</style>