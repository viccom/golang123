<template>
    <div>
        <app-header :user="user" />
        <div class="golang-home-body">
            <div class="golang-home-body-left vote-detail-left">
                <div class="vote-detail-box">
                    <div class="detail-title-box">
                        <p class="vote-detail-title">{{vote.name}}</p>
                        <p class="vote-title-info">
                            <span class="vote-title-info-item">
                                发布于{{vote.createdAt | getReplyTime}}
                            </span>
                            <span class="vote-title-info-item">
                                作者{{vote.user.name}}
                            </span>
                            <span class="vote-title-info-item">
                                {{vote.browseCount}}次浏览
                            </span>
                            <span class="vote-categoties">    {{!isEnd ? '进行中' : '已结束'}}</span>
                        </p>
                    </div>
                    <div class="home-vote-box">
                        <div class="golang123-editor" v-html="vote.content"></div>
                        <div class="">
                            <span v-for="item in vote.voteItems">
                                <Button type="primary" class="vote-item" @click="onVoteSubmit(item.id)">支持<span class="vote-item-label">{{item.name}}</span><span class="vote-item-label">{{item.count}}</span></Button>
                            </span>
                        </div>
                        <div v-if="!isEnd" style="margin-top: 20px;font-size: 14px;font-weight: 700;">{{endAtStr}}&nbsp;&nbsp;前可投票</div>
                        <div class="vote-actions">
                            <div class="vote-share">
                                <div class="vote-share-btn" @click="collect">
                                    <Icon type="android-star-outline" style="font-size: 20px;margin-top:-2px;"></Icon>
                                    <span>收藏</span>
                                </div>
                                <div class="vote-share-btn">
                                    <Icon type="android-share-alt" style="font-size: 16px"></Icon>
                                    <span>分享</span>
                                </div>
                                <template v-if="isAuthor">
                                    <div class="vote-share-btn">
                                        <Icon type="edit" style="font-size: 16px"></Icon>
                                        <a :href="'/vote/edit/' + vote.id"><span>编辑</span></a>
                                    </div>
                                    <div class="vote-share-btn">
                                        <Icon type="android-delete" style="font-size: 17px;"></Icon>
                                        <span @click="onDelete">删除</span>
                                    </div>
                                </template>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="golang-cell comment-box">
                    <div class="title">{{vote.commentCount > 0 ? vote.commentCount : '暂无'}}回复</div>
                    <div class="comment-content">
                        <template v-if="vote.commentCount">
                            <div :id="`reply-${item.id}`" class="comment-item" v-for="(item, index) in vote.comments">
                                <div class="reply-user-icon-box">
                                    <a class="reply-user-icon" :href="`/user/${item.user.id}`" target="_blank">
                                        <img :src="item.user.avatarURL" alt="">
                                    </a>
                                </div>
                                <div class="reply-user-box">
                                    <div>
                                        <a class="reply-user-name" :href="`/user/${item.user.id}`" target="_blank">{{item.user.name}}</a>
                                        <span class="reply-time">{{index + 1}}楼•{{item.createdAt | getReplyTime}}</span>
                                        <div class="comment-actions">
                                            <div v-if="user && user.id !== item.user.id" class="comment-reply" @click="onReplyUser(item)">
                                                <Icon type="reply" style="font-size: 17px;"></Icon>
                                                <span class="comment-reply-txt">回复</span>
                                            </div>
                                            <div v-if="user && user.id === item.user.id" class="comment-delete" @click="onCommentDelete(item.id)">
                                                <Icon type="android-delete" style="font-size: 17px;"></Icon>
                                                <span class="comment-delete-txt">删除</span>
                                            </div>
                                        </div>
                                    </div>
                                    <div v-if="item.parentID" class="parent-comment">
                                        <template v-if="item.parents && item.parents.length">
                                            <span>对</span>
                                            <span><a :href="`/user/${item.parents[0].user.id}`" target="_blank"><img :src="item.parents[0].user.avatarURL"></a></span>
                                            <span><a :href="`/user/${item.parents[0].user.id}`" target="_blank" class="parent-comment-user">{{item.parents[0].user.name}}</a></span>
                                            <span><a :href="`/vote/${vote.id}#reply-${item.parents[0].id}`">{{floorMap[item.parents[0].id]}}楼</a></span>
                                            <span>回复</span>
                                        </template>
                                        <template v-else>
                                            <span style="text-decoration: line-through;">此回复已被作者删除</span>
                                        </template>
                                    </div>

                                    <div class="golang123-editor" v-html="item.content"></div>
                                    <div v-if="item.replyVisible">
                                        <div>
                                            <md-editor :user="user" :value="formData.content" @change="onContentChage" />
                                        </div>
                                        <Row>
                                            <Button @click="onSubmitReply" type="primary">保存</Button>
                                            <Button @click="cancelReplyUser" style="margin-left: 10px;" type="ghost">取消</Button>
                                        </Row>
                                    </div>
                                </div>
                            </div>
                        </template>
                        <p class="not-signin" v-if="!vote.commentCount && user">暂时还没有人回复过这个投票</p>
                        <p class="not-signin" v-if="!vote.commentCount && !user">暂时还没有人回复过这个投票,&nbsp;要回复投票, 请先&nbsp;<a @click="onSignin">登录</a>&nbsp;或&nbsp;<a href="/signup">注册</a></p>
                        <p class="not-signin not-signin-border" v-if="vote.commentCount && !user">要回复投票, 请先&nbsp;<a @click="onSignin">登录</a>&nbsp;或&nbsp;<a href="/signup">注册</a></p>
                    </div>
                </div>
                <div class="golang-cell comment-box" v-if="user && replyVote">
                    <div class="title">添加回复</div>
                    <div class="comment-content">
                        <Form ref="formData" :model="formData" :rules="formRule">
                            <Form-item prop="content">
                                <md-editor :user="user" :value="formData.content" @change="onContentChage" />
                            </Form-item>
                        </Form>
                        <Button type="primary" @click="onSubmitReply">发表回复</Button>
                    </div>
                </div>
            </div>
            <app-sidebar :score="score" :votesMaxBrowse="votesMaxBrowse" :votesMaxComment="votesMaxComment"/>
        </div>
        <app-footer />
        <Modal
            v-model="collectShow"
            class="collect-modal"
            title="添加收藏"
            @on-cancel="cancel">
            <Row
                class="not-signin-dividing collect-row"
                type="flex"
                justify="space-between"
                align="middle"
                v-for="(item, index) in collectDirList" key="index">
                <div>
                    <a :href="`/user/collect/${user.id}?folder=${item.id}`" target="_blank" class="collects-item-label">{{item.name}}</a>
                    <p class="collects-item-num">{{(item.collects && item.collects.length) || 0}}条内容</p>
                </div>
                <Button v-if="item.hasCollect" class="info-button" style="width: 80px" disabled="disabled">已收藏</Button>
                <Button v-else class="info-button" style="width: 80px" @click="createCollect(item.id)">收藏</Button>
            </Row>
            <Button
                type="primary"
                size="large"
                class="collect-dir-btn" @click="createCollectDir">创建收藏夹</Button>
            <div slot="footer"></div>
        </Modal>
        <Modal
            v-model="collectShowDir"
            class="collect-modal"
            title="创建新收藏夹"
            @on-cancel="cancel">
            <Form
                ref="CollectDir"
                :model="collectData"
                :rules="collectRule">
                <Form-item prop="title">
                    <i-input
                        v-model="collectData.title"
                        placeholder="收藏夹名称"
                        size="large"/>
                </Form-item>
            </Form>
            <Row type="flex" justify="space-between">
                <Button type="ghost" style="width:48%" @click="collect">返回</Button>
                <Button type="primary" style="width:48%" @click="submitCollectDir">确认创建</Button>
            </Row>
            <div slot="footer"></div>
        </Modal>
        <BackTop></BackTop>
    </div>
</template>

<script>
    import ErrorCode from '~/constant/ErrorCode'
    import config from '~/config'
    import VoteStatus from '~/constant/VoteStatus'
    import Header from '~/components/Header'
    import Footer from '~/components/Footer'
    import Sidebar from '~/components/Sidebar'
    import Editor from '~/components/Editor'
    import request from '~/net/request'
    import dateTool from '~/utils/date'
    import {trim} from '~/utils/tool'

    export default {
        data () {
            return {
                collectShowDir: false,
                collectShow: false,
                loading: false,
                formData: {
                    content: ''
                },
                formRule: {
                    content: [
                        { required: true, message: '请输入回复内容', trigger: 'blur' }
                    ]
                },
                collectData: {
                    title: ''
                },
                collectRule: {
                    title: [
                        { required: true, message: '请输入收藏夹名称', trigger: 'blur' }
                    ]
                }
            }
        },
        validate ({ params }) {
            var hasId = !!params.id
            return hasId
        },
        asyncData (context) {
            let arr = [
                request.getVote({
                    client: context.req,
                    params: {
                        id: context.params.id
                    }
                }),
                request.getVoteMaxBrowse({
                    client: context.req
                }),
                request.getVoteMaxComment({
                    client: context.req
                }),
                request.getTop10({
                    client: context.req
                })
            ]
            if (context.user) {
                arr.push(request.getFoldersSource({
                    client: context.req
                }))
            }
            return Promise.all(arr).then(arr => {
                let vote = arr[0].data
                let votesMaxBrowse = arr[1].data.votes
                let votesMaxComment = arr[2].data.votes
                let score = arr[3].data.users
                let isAuthor = context.user && context.user.id === vote.user.id
                let collectDirList = []
                if (arr[4]) {
                    collectDirList = arr[4].data.folders || []
                    collectDirList.map(item => {
                        item.hasCollect = false
                        item.collects.map(items => {
                            if (items.sourceID === parseInt(context.params.id) && items.sourceName === 'collect_source_vote') {
                                item.hasCollect = true
                            }
                        })
                    })
                }
                vote.comments = vote.comments || []
                let floorMap = {}
                for (let i = 0; i < vote.comments.length; i++) {
                    vote.comments[i].replyVisible = false
                    floorMap[vote.comments[i].id] = i + 1
                }
                return {
                    isAuthor: isAuthor,
                    vote: vote,
                    floorMap: floorMap,
                    endAtStr: dateTool.formatYMDHM2(vote.endAt),
                    user: context.user,
                    replyVote: true, // 直接回复投票的编辑器是否显示(即parentCommentID为0)
                    parentCommentID: 0,
                    votesMaxBrowse: votesMaxBrowse,
                    votesMaxComment: votesMaxComment,
                    score: score,
                    isEnd: vote.status !== VoteStatus.VOTE_UNDERWAY,
                    collectDirList: collectDirList
                }
            }).catch(err => {
                console.log(err)
                context.error({ statusCode: 404, message: 'Page not found' })
            })
        },
        middleware: 'userInfo',
        methods: {
            onSignin () {
                location.href = '/signin?ref=' + encodeURIComponent(location.href)
            },
            onDelete () {
                let self = this
                this.$Modal.confirm({
                    title: '删除投票',
                    content: '确认删除这个投票?',
                    onOk () {
                        request.deleteVote({
                            params: {
                                id: self.vote.id
                            }
                        }).then(res => {
                            if (res.errNo === ErrorCode.SUCCESS) {
                                self.$Message.success('已删除!')
                                setTimeout(function () {
                                    location.href = '/vote'
                                }, 500)
                            } else {
                                self.$Message.error(res.msg)
                            }
                        }).catch(err => {
                            err = '内部错误'
                            self.$Message.error(err)
                        })
                    },
                    onCancel () {

                    }
                })
            },
            onCommentDelete (id) {
                let self = this
                this.$Modal.confirm({
                    title: '删除回复',
                    content: '确定要删除这个回复?',
                    onOk () {
                        request.deleteComment({
                            params: {
                                id: id
                            }
                        }).then((res) => {
                            if (res.errNo === ErrorCode.SUCCESS) {
                                self.$Message.success({
                                    duration: config.messageDuration,
                                    closable: true,
                                    content: '回复已删除'
                                })
                                return request.getSiteComments({
                                    params: {
                                        sourceID: self.$route.params.id,
                                        sourceName: 'vote'
                                    }
                                })
                            } else {
                                return Promise.reject(new Error(res.msg))
                            }
                        }).then(res => {
                            if (res.errNo === ErrorCode.SUCCESS) {
                                let comments = res.data.comments || []
                                let floorMap = {}
                                for (let i = 0; i < comments.length; i++) {
                                    comments[i].replyVisible = false
                                    floorMap[comments[i].id] = i + 1
                                }
                                self.vote.comments = comments
                                self.vote.commentCount = comments.length
                                self.replyVote = true
                                self.floorMap = floorMap
                            }
                        }).catch(err => {
                            self.$Message.error(err.message)
                        })
                    },
                    onCancel () {

                    }
                })
            },
            onContentChage (content) {
                this.formData.content = content
            },
            onReplyUser (comment) {
                // 对回复进行回复
                let commentID = comment.id
                for (let i = 0; i < this.vote.comments.length; i++) {
                    this.vote.comments[i].replyVisible = false
                    if (this.vote.comments[i].id === commentID) {
                        this.vote.comments[i].replyVisible = true
                    }
                }
                this.parentCommentID = commentID
                this.replyVote = false
            },
            cancelReplyUser () {
                for (let i = 0; i < this.vote.comments.length; i++) {
                    this.vote.comments[i].replyVisible = false
                }
                this.parentCommentID = 0
                this.replyVote = true
            },
            onSubmitReply () {
                // 验证交给后台
                let commentID
                if (!this.loading) {
                    this.loading = true
                    request.commentCreate({
                        body: {
                            sourceID: parseInt(this.$route.params.id),
                            parentID: this.parentCommentID,
                            content: this.formData.content,
                            sourceName: 'vote'
                        }
                    }).then(res => {
                        if (res.errNo === ErrorCode.SUCCESS) {
                            commentID = res.data.comment.id
                            this.formData.content = ''
                            this.$Message.success({
                                duration: config.messageDuration,
                                closable: true,
                                content: '回复提交成功'
                            })
                            return request.getSiteComments({
                                params: {
                                    sourceID: this.$route.params.id,
                                    sourceName: 'vote'
                                }
                            })
                        } else {
                            return Promise.reject(new Error(res.msg))
                        }
                    }).then(res => {
                        if (res.errNo === ErrorCode.SUCCESS) {
                            let comments = res.data.comments || []
                            let floorMap = {}
                            for (let i = 0; i < comments.length; i++) {
                                comments[i].replyVisible = false
                                floorMap[comments[i].id] = i + 1
                            }
                            this.vote.comments = comments
                            this.vote.commentCount = comments.length
                            this.replyVote = true
                            this.parentCommentID = 0
                            this.floorMap = floorMap
                            location.href = `/vote/${this.vote.id}#reply-${commentID}`
                            setTimeout(() => {
                                let replyDOM = document.getElementById(`reply-${commentID}`)
                                replyDOM.scrollIntoView && replyDOM.scrollIntoView()
                            }, 100)
                        }
                        this.loading = false
                    }).catch(err => {
                        this.loading = false
                        this.$Message.error({
                            duration: config.messageDuration,
                            closable: true,
                            content: err.message
                        })
                    })
                }
            },
            onVoteSubmit (id) {
                if (!this.loading) {
                    this.loading = true
                    request.userVote({
                        params: {
                            id: id
                        }
                    }).then(res => {
                        this.loading = false
                        if (res.errNo === ErrorCode.SUCCESS) {
                            return request.getVote({
                                params: {
                                    id: this.$route.params.id
                                }
                            })
                        } else {
                            return Promise.reject(new Error(res.msg))
                        }
                    }).then(res => {
                        if (res.errNo === ErrorCode.SUCCESS) {
                            this.vote = res.data
                            this.$Message.success({
                                duration: config.messageDuration,
                                closable: true,
                                content: '投票成功'
                            })
                        }
                    }).catch(err => {
                        this.loading = false
                        this.$Message.warning({
                            duration: config.messageDuration,
                            closable: true,
                            content: err.message
                        })
                    })
                }
            },
            collect () {
                if (!this.user) {
                    window.location.href = '/signin'
                }
                this.collectShowDir = false
                this.collectData.title = ''
                this.collectShow = true
            },
            cancel () {
                this.collectShowDir = false
                this.collectShow = false
                this.collectData.title = ''
            },
            createCollectDir () {
                this.collectShowDir = true
                this.collectShow = false
            },
            submitCollectDir () {
                this.$refs['CollectDir'].validate(valid => {
                    if (!this.loading && valid) {
                        this.loading = true
                        request.createCollectDir({
                            body: {
                                name: trim(this.collectData.title),
                                parentID: 0
                            }
                        }).then(res => {
                            this.loading = false
                            if (res.errNo === ErrorCode.SUCCESS) {
                                res.data.hasCollect = false
                                this.collectDirList.unshift(res.data)
                                this.collect()
                            } else {
                                this.$Message.error(res.msg)
                            }
                        }).catch(err => {
                            this.loading = false
                            this.$Message.error(err.message)
                        })
                    }
                })
            },
            createCollect (id) {
                if (this.loading) {
                    return
                }
                this.loading = true
                request.createCollect({
                    body: {
                        sourceName: 'collect_source_vote',
                        sourceID: parseInt(this.$route.params.id),
                        folderID: id
                    }
                }).then(res => {
                    this.loading = false
                    if (res.errNo === ErrorCode.SUCCESS) {
                        this.collectDirList.map(item => {
                            if (item.id === id) {
                                item.hasCollect = true
                            }
                        })
                        console.log(this.collectDirList)
                    } else {
                        this.$Message.error(res.msg)
                    }
                }).catch(err => {
                    this.loading = false
                    this.$Message.error(err.message)
                })
            }
        },
        mounted () {
            if (window.location.search && window.location.search.indexOf('pushLink=1') >= 0) {
                var bp = document.createElement('script')
                var curProtocol = window.location.protocol.split(':')[0]
                if (curProtocol === 'https') {
                    bp.src = 'https://zz.bdstatic.com/linksubmit/push.js'
                } else {
                    bp.src = 'http://push.zhanzhang.baidu.com/push.js'
                }
                var s = document.getElementsByTagName('script')[0]
                s.parentNode.insertBefore(bp, s)
            }
        },
        head () {
            return {
                title: this.vote.name,
                link: [
                    { rel: 'stylesheet', href: '/styles/editor/simplemde.min.css' }
                ]
            }
        },
        filters: {
            getReplyTime: dateTool.getReplyTime
        },
        components: {
            'app-header': Header,
            'app-footer': Footer,
            'app-sidebar': Sidebar,
            'md-editor': Editor
        }
    }
</script>

<style>
    @import '../../assets/styles/vote/detail.css'
</style>
