<template>
    <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" :visible.sync="visible">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" label-width="80px">
            <el-form-item label="规则名称" prop="ruleName">
                <el-input v-model="dataForm.ruleName" placeholder="规则名称（若为全局规则建议命名为：全局规则-xxx）"></el-input>
            </el-form-item>
            <el-form-item label="匹配词" prop="matchValue" v-if="!dataForm.globalRule">
                <tags-editor v-model="dataForm.matchValue"></tags-editor>
            </el-form-item>
            <el-row>
                <el-col :span="8">
                    <el-form-item label="精确匹配" prop="exactMatch">
                        <el-switch v-model="dataForm.exactMatch" :active-value="true" :inactive-value="false"></el-switch>
                    </el-form-item>
                </el-col>
                <el-col :span="8">
                    <el-form-item label="是否启用" prop="status">
                        <el-switch v-model="dataForm.status" :active-value="true" :inactive-value="false"></el-switch>
                    </el-form-item>
                </el-col>
                <el-col :span="8">
                    <el-form-item label="全局规则" prop="globalRule">
                        <el-switch v-model="dataForm.globalRule" :active-value="true" :inactive-value="false"></el-switch>
                    </el-form-item>
                </el-col>
            </el-row>
            <el-row>
                <el-col :span="12">
                    <el-form-item label="回复类型" prop="replyType">
                        <el-select v-model="dataForm.replyType" @change="onReplyTypeChange">
                            <el-option v-for="(name,key) in KefuMsgType" :key="key" :value="key" :label="name"></el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-tooltip class="item" effect="dark" content="数字越大，越晚发送" placement="top">
                        <el-form-item label="优先级" prop="priority">
                            <el-input-number v-model="dataForm.priority" size="small" :min="0"></el-input-number>
                        </el-form-item>
                    </el-tooltip>
                </el-col>
            </el-row>
            <el-form-item label="回复内容" prop="replyContent" v-if="dataForm.replyType !== 'news'">
                <el-input v-model="dataForm.replyContent" type="textarea" :rows="5" placeholder="文本、图文ID、media_id、json配置"></el-input>
                <el-button type="text" v-show="'text'==dataForm.replyType" @click="addLink">插入链接</el-button>
                <el-button type="text" v-show="assetsType" @click="assetsSelectorVisible=true">
                    从素材库中选择<span v-if="'miniprogrampage'==dataForm.replyType || 'music'==dataForm.replyType">缩略图</span>
                </el-button>
            </el-form-item>
            <el-row v-if="dataForm.replyType === 'news'">
                <el-divider>文章详情</el-divider>
                <el-form :model="newsForm" :rules="newsRule" ref="newsForm" label-width="80px">
                    <el-form-item label="文章标题" prop="title">
                        <el-input v-model="newsForm.title" placeholder="文章标题"></el-input>
                    </el-form-item>
                    <el-form-item label="文章简介" prop="description">
                        <el-input v-model="newsForm.description" type="textarea" :rows="3" placeholder="文章简介"></el-input>
                    </el-form-item>
                    <el-form-item label="链接地址" prop="url">
                        <el-input v-model="newsForm.url" placeholder="链接地址"></el-input>
                    </el-form-item>
                    <el-form-item label="图片地址" prop="picUrl">
                        <el-input v-model="newsForm.picUrl" placeholder="图片地址"></el-input>
                    </el-form-item>
                </el-form>
            </el-row>
            <el-collapse v-model="activeNames">
                <el-collapse-item title="其他设置" name="other">
                    <el-row>
                        <el-col :span="12">
                            <el-form-item label="生效时间" prop="effectTimeStart">
                                <el-time-picker v-model="dataForm.effectTimeStart" value-format="HH:mm:ss"></el-time-picker>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="失效时间" prop="effectTimeEnd">
                                <el-time-picker v-model="dataForm.effectTimeEnd" value-format="HH:mm:ss"></el-time-picker>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="备注说明" prop="desc">
                        <el-input v-model="dataForm.desc" placeholder="备注说明"></el-input>
                    </el-form-item>
                    <el-row>
                        <el-col :span="12">
                            <el-form-item label="作用范围" prop="appid">
                                <el-select v-model="dataForm.appid" placeholder="作用范围">
                                    <el-option label="当前公众号" :value="selectedAppid"></el-option>
                                    <!-- <el-option label="全部公众号" value=""></el-option> -->
                                </el-select>
                            </el-form-item>
                        </el-col>
                    </el-row>
                </el-collapse-item>
            </el-collapse>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="visible = false">取消</el-button>
            <el-button type="primary" @click="dataFormSubmit()" :loading="submitting">确定</el-button>
        </span>
        <assets-selector v-if="assetsSelectorVisible && assetsType" :visible="assetsSelectorVisible" :selectType="assetsType" @selected="onAssetsSelect"></assets-selector>
    </el-dialog>
</template>

<script>
import { mapState } from 'vuex'
export default {
    components: {
        tagsEditor: () => import('@/components/tags-editor'),
        AssetsSelector:()=>import('./assets/assets-selector')
    },
    data() {
        return {
            activeNames: [],
            visible: false,
            submitting: false,
            assetsSelectorVisible:false,
            dataForm: {
                ruleId: 0,
                appid:'',
                ruleName: "",
                globalRule: false,
                exactMatch: true,
                priority: 0,
                matchValue: "",
                replyType: 'text',
                replyContent: "",
                status: true,
                desc: "",
                effectTimeStart: "00:00:00",
                effectTimeEnd: "23:59:59"
            },
            newsForm: {
                title: "",
                description: "",
                url: "",
                picUrl: ""
            },
            dataRule: {
                ruleName: [
                    { required: true, message: "规则名称不能为空", trigger: "blur" }
                ],
                matchValue: [
                    { required: true, message: "匹配的关键词、事件等不能为空", trigger: "blur" }
                ],
                replyType: [
                    { required: true, message: "回复类型（1:文本2:图文3媒体）不能为空", trigger: "blur" }
                ],
                replyContent: [
                    { required: true, message: "回复内容不能为空", trigger: "blur" }
                ],
                status: [
                    { required: true, message: "是否有效不能为空", trigger: "blur" }
                ],
                exactMatch: [
                    { required: true, message: "是否精确匹配不能为空", trigger: "blur" }
                ],
                globalRule: [
                    { required: true, message: "是否全局规则不能为空", trigger: "blur" }
                ],
                priority: [
                    { required: true, message: "是否全局规则不能为空", trigger: "blur" }
                ],
                effectTimeStart: [
                    { required: true, message: "生效起始时间不能为空", trigger: "blur" }
                ],
                effectTimeEnd: [
                    { required: true, message: "生效结束时间不能为空", trigger: "blur" }
                ]
            },
            newsRule: {
                title: [
                    { required: true, message: "文章标题不能为空", trigger: "blur" }
                ],
                url: [
                    { required: true, message: "文章链接不能为空", trigger: "blur" }
                ]
            }
        };
    },
    mounted() {
        this.dataForm.appid = this.$store.state.wxAccount.selectedAppid;
    },
    computed: mapState({
        KefuMsgType: state=>state.message.KefuMsgType,
        selectedAppid: state=>state.wxAccount.selectedAppid,
        assetsType(){
            const config={//消息类型与选择素材类型对应关系
                'image':'image',
                'voice':'voice',
                'video':'video',
                'mpnews':'news',
                'miniprogrampage':'image',//小程序需选择卡片图
                'music':'image'
            }
            return config[this.dataForm.replyType] || ''
        }
    }),
    methods: {
        init(id) {
            this.dataForm.ruleId = id || 0;
            this.visible = true;
            this.$nextTick(() => {
                this.$refs["dataForm"].resetFields();
                if (this.dataForm.ruleId) {
                    this.$http({
                        url: this.$http.adornUrl( `/manage/msgReplyRule/info/${this.dataForm.ruleId}` ),
                        method: "get",
                        params: this.$http.adornParams()
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            if (data.msgReplyRule !== null) {
                                Object.keys(this.dataForm).forEach(key => {
                                    this.dataForm[key] = data.msgReplyRule[key];
                                });
                            }
                            if (data.msgReplyRule.replyType === 'news') {
                                Object.assign(this.newsForm, JSON.parse(data.msgReplyRule.replyContent));
                            }
                        }
                    });
                }
            });
        },
        // 表单提交
        dataFormSubmit() {
            if (this.submitting) return;
            this.$refs["dataForm"].validate(valid => {
                if (valid) {
                    if (this.dataForm.replyType === 'news') {
                        this.dataForm.replyContent = JSON.stringify(this.newsForm);
                    }
                    this.submitting = true;
                    this.$http({
                        url: this.$http.adornUrl(`/manage/msgReplyRule/${!this.dataForm.ruleId ? "save" : "update"}`),
                        method: "post",
                        data: this.$http.adornData(this.dataForm)
                    }).then(({ data }) => {
                        if (data && data.code === 200) {
                            this.$message({
                                message: "操作成功",
                                type: "success",
                                duration: 1500
                            });
                            this.visible = false;
                            this.$emit("refreshDataList");
                        } else {
                            this.$message.error(data.msg);
                        }
                    }).finally(() => {
                        this.submitting = false;
                    });
                }
            });
        },
        addLink() {
            this.dataForm.replyContent += '<a href="链接地址">链接文字</a>'
        },
        onReplyTypeChange(value) {
            if ("miniprogrampage" == value) {
                let demo = { title: "标题", appid: "小程序APPID", pagepath: "页面地址", thumb_media_id: "缩略图media_id" };
                this.dataForm.replyContent = JSON.stringify(demo, null, 4)
            } else if ("music" == value) {
                let demo = { musicurl: "音乐链接", hqmusicurl: "高品质链接", title: "标题", description: "描述", thumb_media_id: "缩略图media_id" }
                this.dataForm.replyContent = JSON.stringify(demo, null, 4)
            } else if ("msgmenu" == value) {
                let demo = { head_content: "开头文字", list: [{ id: "菜单1ID", content: "菜单2内容" }, { id: "菜单2ID", content: "菜单2内容" }, { id: "菜单nID", content: "菜单n内容" }], tail_content: "结尾文字" }
                this.dataForm.replyContent = JSON.stringify(demo, null, 4)
            } else if ("news" == value) {
                let demo={title:"文章标题",description:"文章简介",url:"链接URL",picUrl:"缩略图URL"}
                this.dataForm.replyContent = JSON.stringify(demo, null, 4)
            } else {
                this.dataForm.replyContent = '媒体素材media_id'
            }
        },
        onAssetsSelect(assetsInfo){
            if(this.dataForm.replyType=='miniprogrampage' || this.dataForm.replyType=='music'){
                let data = JSON.parse(this.dataForm.replyContent)
                if(data && data.thumb_media_id)data.thumb_media_id=assetsInfo.mediaId
                this.dataForm.replyContent = JSON.stringify(data, null, 4)
            }else{
                this.dataForm.replyContent = assetsInfo.mediaId
            }
            this.assetsSelectorVisible=false
        }
    }
};
</script>