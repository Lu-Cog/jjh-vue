<template>
    <div>
        <div class="i-layout-page-header">
            <div class="i-layout-page-header">
                <router-link :to="{path:'/admin/marketing/store_bargain/index'}"><Button icon="ios-arrow-back" size="small"  class="mr20">返回</Button></router-link>
                <span class="ivu-page-header-title mr20" v-text="$route.params.id!=='0'?'编辑砍价商品':'添加砍价商品'"></span>
            </div>
        </div>
        <Card :bordered="false" dis-hover class="ivu-mt">
            <Row type="flex" class="mt30 acea-row row-middle row-center">
                <Col span="20">
                    <Steps :current="current">
                        <Step title="选择砍价商品"></Step>
                        <Step title="填写基础信息"></Step>
                        <Step title="修改商品详情"></Step>
                    </Steps>
                </Col>
                <Col span="23">
                    <Form class="form mt30" ref="formValidate" :rules="ruleValidate" :model="formValidate"  @on-validate="validate"
                          :label-width="labelWidth" :label-position="labelPosition" @submit.native.prevent>
                        <FormItem label="选择商品：" prop="image_input" v-show="current === 0">
                            <div class="picBox" @click="changeGoods">
                                <div class="pictrue" v-if="formValidate.image"><img
                                        v-lazy="formValidate.image"></div>
                                <div class="upLoad acea-row row-center-wrapper" v-else>
                                    <Icon type="ios-camera-outline" size="26" class="iconfonts"/>
                                </div>
                            </div>
                        </FormItem>
                        <Row v-show="current === 1" type="flex">
                            <Col span="24">
                                <FormItem label="商品主图：" prop="image">
                                    <div class="picBox" @click="modalPicTap('dan','danFrom')">
                                        <div class="pictrue" v-if="formValidate.image"><img v-lazy="formValidate.image"></div>
                                        <div class="upLoad acea-row row-center-wrapper" v-else>
                                            <Icon type="ios-camera-outline" size="26" class="iconfonts"/>
                                        </div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="商品轮播图：" prop="images">
                                    <div class="acea-row">
                                        <div class="pictrue" v-for="(item,index) in formValidate.images" :key="index"
                                             draggable="true"
                                             @dragstart="handleDragStart($event, item)"
                                             @dragover.prevent="handleDragOver($event, item)"
                                             @dragenter="handleDragEnter($event, item)"
                                             @dragend="handleDragEnd($event, item)">
                                            <img v-lazy="item">
                                            <Button shape="circle" icon="md-close"  @click.native="handleRemove(index)" class="btndel"></Button>
                                        </div>
                                        <div v-if="formValidate.images.length<10" class="upLoad acea-row row-center-wrapper" @click="modalPicTap('duo')">
                                            <Icon type="ios-camera-outline" size="26" class="iconfonts"/>
                                        </div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <Col v-bind="grid">
                                    <FormItem label="砍价活动名称：" prop="title" label-for="title">
                                        <Input placeholder="请输入砍价活动名称" element-id="title" v-model="formValidate.title" />
                                    </FormItem>
                                </Col>
                            </Col>
                            <Col span="24">
                                <Col v-bind="grid">
                                    <FormItem label="砍价活动简介：" prop="info" label-for="info">
                                        <Input placeholder="请输入砍价活动简介" type="textarea" :rows="4" element-id="info" v-model="formValidate.info"/>
                                    </FormItem>
                                </Col>
                            </Col>
                            <Col span="24">
                                <FormItem label="活动时间：" prop="section_time">
                                    <div class="acea-row row-middle">
                                        <DatePicker :editable="false" type="datetimerange" format="yyyy-MM-dd HH:mm" placeholder="请选择活动时间"
                                                    @on-change="onchangeTime" class="perW30" :value="formValidate.section_time" v-model="formValidate.section_time"></DatePicker>
                                        <div class="ml10 grey">设置活动开启结束时间，用户可以在设置时间内发起参与砍价</div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="运费模板：" prop="temp_id">
                                    <div class="acea-row row-middle">
                                        <Select v-model="formValidate.temp_id" class="perW35">
                                            <Option v-for="item in templateList" :value="item.id" :key="item.id">{{ item.name }}</Option>
                                        </Select>
                                        <div class="ml10 grey"><router-link :to="{path:'/admin/setting/freight/shipping_templates/list'}">添加运费模板</router-link></div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="砍价人数：" prop="people_num" label-for="people_num">
                                    <div class="acea-row row-middle">
                                        <InputNumber  placeholder="请输入砍价人数" element-id="people_num" :min="2" :precision="0" v-model="formValidate.people_num"  class="perW20"/>
                                        <div class="ml10 grey">需要多少人砍价成功</div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="帮砍次数：" prop="bargain_num" label-for="bargain_num">
                                    <div class="acea-row row-middle">
                                        <InputNumber  placeholder="请输入帮砍次数" element-id="bargain_num" :min="1" :precision="0" v-model="formValidate.bargain_num" class="perW20"/>
                                        <div class="ml10 grey">单个商品用户可以帮砍的次数，例：次数设置为1，甲和乙同时将商品A的砍价链接发给丙，丙只能帮甲或乙其中一个人砍价</div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="购买数量限制：" prop="num">
                                    <div class="acea-row row-middle">
                                        <InputNumber placeholder="购买数量限制" :precision="0" :min="1"  v-model="formValidate.num" class="perW20"/>
                                        <div class="ml10 grey">单个活动每个用户发起砍价次数限制</div>
                                    </div>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="单位：" prop="unit_name" label-for="unit_name">
                                    <Input placeholder="请输入单位" element-id="unit_name" v-model="formValidate.unit_name" class="perW20"/>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="排序：">
                                    <InputNumber placeholder="请输入排序" element-id="sort" :precision="0"  v-model="formValidate.sort" class="perW10"/>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="活动状态：" props="status" label-for="status">
                                    <RadioGroup element-id="status"  v-model="formValidate.status">
                                        <Radio :label="1" class="radio">开启</Radio>
                                        <Radio :label="0">关闭</Radio>
                                    </RadioGroup>
                                </FormItem>
                            </Col>
                            <Col span="24">
                                <FormItem label="规格选择：">
                                    <Table :data="specsData" :columns="columns" border>
                                        <template slot-scope="{ row, index }" slot="pic">
                                            <div class="acea-row row-middle row-center-wrapper" @click="modalPicTap('dan','danTable',index)">
                                                <div class="pictrue pictrueTab" v-if="specsData[index].pic"><img v-lazy="specsData[index].pic"></div>
                                                <div class="upLoad pictrueTab acea-row row-center-wrapper"   v-else>
                                                    <Icon type="ios-camera-outline" size="21" class="iconfonts"/>
                                                </div>
                                            </div>
                                        </template>
                                        <template slot-scope="{ row, index }" slot="price">
                                            <InputNumber v-model="specsData[index].price" :min="0" active-change class="priceBox"></InputNumber>
                                        </template>
                                        <template slot-scope="{ row, index }" slot="min_price">
                                            <InputNumber v-model="specsData[index].min_price" :min="0" active-change class="priceBox"></InputNumber>
                                        </template>
                                        <template slot-scope="{ row, index }" slot="quota">
                                            <InputNumber v-model="specsData[index].quota" :min="1" active-change class="priceBox"></InputNumber>
                                        </template>
                                    </Table>
                                </FormItem>
                            </Col>
                        </Row>
                        <div v-show="current === 2">
                            <FormItem label="内容：">
                                <vue-ueditor-wrap v-model="formValidate.description" :key="1"   @beforeInit="addCustomDialog"   :config="myConfig"  style="width: 90%;"></vue-ueditor-wrap>
                            </FormItem>
                            <FormItem label="规则：">
                                <vue-ueditor-wrap v-model="formValidate.rule" :key="2"   @beforeInit="addCustomDialog"  :config="myConfig"  style="width: 90%;"></vue-ueditor-wrap>
                            </FormItem>
                        </div>
                        <FormItem>
                            <Button class="submission mr15" @click="step" v-show="current!==0" :disabled="$route.params.id&&$route.params.id!=='0'&&current===1">上一步</Button>
                            <Button type="primary" :disabled="submitOpen && current === 2" class="submission" @click="next('formValidate')" v-text="current===2?'提交':'下一步'"></Button>
                        </FormItem>
                        <Spin size="large" fix v-if="spinShow"></Spin>
                    </Form>
                </Col>
            </Row>
        </Card>
        <!-- 选择商品-->
        <Modal v-model="modals" title="商品列表" footerHide  class="paymentFooter" scrollable width="900" @on-cancel="cancel">
            <goods-list ref="goodslist"  @getProductId="getProductId"></goods-list>
        </Modal>

        <!-- 上传图片-->
        <Modal v-model="modalPic" width="950px" scrollable  footer-hide closable title='上传商品图' :mask-closable="false" :z-index="888">
            <uploadPictures :isChoice="isChoice" @getPic="getPic"  @getPicD="getPicD" :gridBtn="gridBtn" :gridPic="gridPic" v-if="modalPic"></uploadPictures>
        </Modal>
    </div>
</template>

<script>
    import { mapState } from 'vuex';
    import goodsList from '@/components/goodsList/index';
    import uploadPictures from '@/components/uploadPictures';
    import { bargainInfoApi, bargainCreatApi, productAttrsApi } from '@/api/marketing';
    import { productGetTemplateApi } from '@/api/product';
    import VueUeditorWrap from 'vue-ueditor-wrap';
    export default {
        name: 'storeBargainCreate',
        components: { goodsList, uploadPictures, VueUeditorWrap },
        data () {
            return {
                submitOpen: false,
                spinShow: false,
                myConfig: {
                    autoHeightEnabled: false, // 编辑器不自动被内容撑高
                    initialFrameHeight: 500, // 初始容器高度
                    initialFrameWidth: '100%', // 初始容器宽度
                    UEDITOR_HOME_URL: '/admin/UEditor/',
                    serverUrl: ''
                },
                isChoice: '',
                current: 0,
                modalPic: false,
                grid: {
                    xl: 12,
                    lg: 20,
                    md: 24,
                    sm: 24,
                    xs: 24
                },
                grid2: {
                    xl: 8,
                    lg: 8,
                    md: 12,
                    sm: 24,
                    xs: 24
                },
                gridPic: {
                    xl: 6,
                    lg: 8,
                    md: 12,
                    sm: 12,
                    xs: 12
                },
                gridBtn: {
                    xl: 4,
                    lg: 8,
                    md: 8,
                    sm: 8,
                    xs: 8
                },
                modals: false,
                modal_loading: false,
                images: [],
                templateList: [],
                columns: [],
                specsData: [],
                formValidate: {
                    images: [],
                    info: '',
                    title: '',
                    store_name: '',
                    image: '',
                    unit_name: '',
                    price: 0,
                    min_price: 0,
                    bargain_max_price: 10,
                    bargain_min_price: 0.01,
                    cost: 0,
                    bargain_num: 1,
                    people_num: 2,
                    stock: 1,
                    sales: 0,
                    sort: 0,
                    num: 1,
                    give_integral: 0,
                    postage: 0,
                    is_postage: 0,
                    is_hot: 0,
                    status: 0,
                    section_time: [],
                    description: '',
                    rule: '',
                    id: 0,
                    product_id: 0,
                    temp_id: '',
                    attrs: [],
                    items: []
                },
                ruleValidate: {
                    image: [
                        { required: true, message: '请选择主图', trigger: 'change' }
                    ],
                    images: [
                        { required: true, type: 'array', message: '请选择主图', trigger: 'change' },
                        { type: 'array', min: 1, message: 'Choose two hobbies at best', trigger: 'change' }
                    ],
                    title: [
                        { required: true, message: '请输入砍价活动名称', trigger: 'blur' }
                    ],
                    info: [
                        { required: true, message: '请输入砍价活动简介', trigger: 'blur' }
                    ],
                    store_name: [
                        { required: true, message: '请输入砍价商品名称', trigger: 'blur' }
                    ],
                    section_time: [
                        { required: true, type: 'array', message: '请选择活动时间', trigger: 'change' }
                    ],
                    unit_name: [
                        { required: true, message: '请输入单位', trigger: 'blur' }
                    ],
                    price: [
                        { required: true, type: 'number', message: '请输入原价', trigger: 'blur' }
                    ],
                    min_price: [
                        { required: true, type: 'number', message: '请输入最低购买价', trigger: 'blur' }
                    ],
                    // bargain_max_price: [
                    //     { required: true, type: 'number', message: '请输单次砍价最大金额', trigger: 'blur' }
                    // ],
                    // bargain_min_price: [
                    //     { required: true, type: 'number', message: '单次砍价最小金额', trigger: 'blur' }
                    // ],
                    cost: [
                        { required: true, type: 'number', message: '请输入成本价', trigger: 'blur' }
                    ],
                    bargain_num: [
                        { required: true, type: 'number', message: '请输入帮砍次数', trigger: 'blur' }
                    ],
                    people_num: [
                        { required: true, type: 'number', message: '请输入砍价人数', trigger: 'blur' }
                    ],
                    stock: [
                        { required: true, type: 'number', message: '请输入库存', trigger: 'blur' }
                    ],
                    num: [
                        { required: true, type: 'number', message: '请输入单次允许购买数量', trigger: 'blur' }
                    ],
                    temp_id: [
                        { required: true, message: '请选择运费模板', trigger: 'change', type: 'number' }
                    ]
                },
                currentid: '',
                picTit: '',
                tableIndex: 0,
                copy:0
            }
        },
        computed: {
            ...mapState('media', [
                'isMobile'
            ]),
            labelWidth () {
                return this.isMobile ? undefined : 140;
            },
            labelPosition () {
                return this.isMobile ? 'top' : 'right';
            }
        },
        mounted () {
            if (this.$route.params.id !== '0' && this.$route.params.id) {
                this.copy = this.$route.params.copy;
                this.current = 1;
                this.getInfo();
            }
            this.productGetTemplate();
        },
        methods: {
            // 砍价规格；
            productAttrs (row) {
                let that = this;
                productAttrsApi(row.id, 2).then(res => {
                    let data = res.data.info;
                    let radio = {
                        title: '选择',
                        key: 'chose',
                        width: 60,
                        align: 'center',
                        render: (h, params) => {
                            let uid = params.index;
                            let flag = false;
                            if (this.currentid === uid) {
                                flag = true
                            } else {
                                flag = false
                            }
                            let self = this;
                            return h('div', [
                                h('Radio', {
                                    props: {
                                        value: flag
                                    },
                                    on: {
                                        'on-change': () => {
                                            self.currentid = uid;
                                            let attrs = [];
                                            attrs.push(params.row);
                                            self.formValidate.attrs = attrs;
                                        }
                                    }
                                })
                            ])
                        }
                    };
                    that.columns = data.header;
                    that.columns.unshift(radio);
                    that.specsData = data.attrs;
                    that.formValidate.items = data.items;
                }).catch(res => {
                    that.$Message.error(res.msg);
                })
            },
            // 获取运费模板；
            productGetTemplate () {
                productGetTemplateApi().then(res => {
                    this.templateList = res.data;
                })
            },
            // 商品id
            getProductId (row) {
                this.modal_loading = false;
                this.modals = false;
                setTimeout(() => {
                    this.formValidate = {
                        images: row.slider_image,
                        info: row.store_info,
                        title: row.store_name,
                        store_name: row.store_name,
                        image: row.image,
                        unit_name: row.unit_name,
                        price: 0, // 不取商品中的原价
                        min_price: 0,
                        bargain_max_price: 10,
                        bargain_min_price: 0.01,
                        cost: row.cost,
                        bargain_num: 1,
                        people_num: 2,
                        stock: row.stock,
                        sales: row.sales,
                        sort: row.sort,
                        num: 1,
                        give_integral: row.give_integral,
                        postage: row.postage,
                        is_postage: row.is_postage,
                        is_hot: row.is_hot,
                        status: 0,
                        section_time: [],
                        description: row.description, // 不取商品中的
                        rule: '',
                        id: 0,
                        product_id: row.id,
                        temp_id: row.temp_id
                    };
                    this.productAttrs(row);
                }, 500);
            },
            cancel () {
                this.modals = false;
            },
            // 移动
            handleDragStart (e, item) {
                this.dragging = item;
            },
            handleDragEnd (e, item) {
                this.dragging = null
            },
            // 首先把div变成可以放置的元素，即重写dragenter/dragover
            handleDragOver (e) {
                e.dataTransfer.dropEffect = 'move'
            },
            handleDragEnter (e, item) {
                e.dataTransfer.effectAllowed = 'move'
                if (item === this.dragging) {
                    return
                }
                const newItems = [...this.formValidate.images]
                const src = newItems.indexOf(this.dragging)
                const dst = newItems.indexOf(item)
                newItems.splice(dst, 0, ...newItems.splice(src, 1))
                this.formValidate.images = newItems;
            },
            // 具体日期
            onchangeTime (e) {
                this.formValidate.section_time = e;
            },
            // 详情
            getInfo () {
                this.spinShow = true;
                bargainInfoApi(this.$route.params.id).then(async res => {
                    let that = this;
                    let info = res.data.info;
                    this.formValidate = info;
                    this.formValidate.rule = info.rule === null ? '' : info.rule;
                    this.$set(this.formValidate, 'items', info.attrs.items);
                    this.columns = info.attrs.header;
                    let radio = {
                        title: '选择',
                        key: 'chose',
                        width: 60,
                        align: 'center',
                        render: (h, params) => {
                            let uid = params.index;
                            let flag = false;
                            if (this.currentid === uid) {
                                flag = true
                            } else {
                                flag = false
                            }
                            let self = this;
                            return h('div', [
                                h('Radio', {
                                    props: {
                                        value: flag
                                    },
                                    on: {
                                        'on-change': () => {
                                            self.currentid = uid;
                                            let attrs = [];
                                            attrs.push(params.row);
                                            self.formValidate.attrs = attrs;
                                        }
                                    }
                                })
                            ])
                        }
                    };
                    that.columns.unshift(radio);
                    this.specsData = info.attrs.value;
                    let defaultAttrs = [];
                    info.attrs.value.forEach(function (item, index) {
                        if (item.opt) {
                            defaultAttrs.push(item);
                            that.$set(that, 'currentid', index);
                            that.$set(that.formValidate, 'attrs', defaultAttrs)
                        }
                    });
                    this.spinShow = false;
                }).catch(res => {
                    this.spinShow = false;
                    this.$Message.error(res.msg);
                })
            },
            // 下一步
            next (name) {
                if (this.current === 2) {
                    this.$refs[name].validate((valid) => {
                        if (valid) {
                            if (this.copy == 1 )this.formValidate.copy = 1;
                            this.formValidate.id = (this.$route.params.id) || 0;
                            this.submitOpen = true;
                            bargainCreatApi(this.formValidate).then(async res => {
                                this.submitOpen = false;
                                this.$Message.success(res.msg);
                                setTimeout(() => {
                                    this.$router.push({ path: '/admin/marketing/store_bargain/index' });
                                }, 500);
                            }).catch(res => {
                                this.submitOpen = false;
                                this.$Message.error(res.msg);
                            })
                        } else {
                            return false;
                        }
                    })
                } else if(this.current === 1){
                    this.$refs[name].validate((valid) => {
                        if(valid){
                            if (this.currentid === '') {
                                return this.$Message.error('请选择属性规格');
                            } else {
                                let val = this.specsData[this.currentid];
                                let formValidate = this.formValidate.attrs[0];
                                formValidate.price = val.price;
                                formValidate.min_price = val.min_price;
                                formValidate.quota = val.quota;
                                if (this.formValidate.attrs[0].quota <= 0) {
                                    return this.$Message.error('砍价限量必须大于0');
                                }
                            }
                            this.current += 1;
                        }else {
                            return this.$Message.warning('请完善您的信息');
                        }
                    });
                } else {
                    if (this.formValidate.image) {
                        this.current += 1;
                    } else {
                        this.$Message.warning('请选择商品');
                    }
                }
            },
            // 上一步
            step () {
                this.current--;
            },
            // 内容
            getContent (val) {
                this.formValidate.description = val;
            },
            // 规则
            getRole (val) {
                this.formValidate.rule = val;
            },
            // 点击商品图
            modalPicTap (tit, picTit, index) {
                this.modalPic = true;
                this.isChoice = tit === 'dan' ? '单选' : '多选';
                this.picTit = picTit;
                this.tableIndex = index;
            },
            // 获取单张图片信息
            getPic (pc) {
                switch (this.picTit) {
                    case 'danFrom':
                        this.formValidate.image = pc.att_dir;
                        break;
                    default:
                        this.specsData[this.tableIndex].pic = pc.att_dir;
                        this.formValidate.attrs[0].pic = pc.att_dir;
                }
                this.modalPic = false;
            },
            // 获取多张图信息
            getPicD (pc) {
                this.images = pc;
                this.images.map((item) => {
                    this.formValidate.images.push(item.att_dir)
                    this.formValidate.images = this.formValidate.images.splice(0,10);
                });
                this.modalPic = false;
            },
            handleRemove (i) {
                this.images.splice(i, 1);
                this.formValidate.images.splice(i, 1);
                console.log(this.images);
            },
            // 选择商品
            changeGoods () {
                this.modals = true;
                this.$refs.goodslist.getList();
                this.$refs.goodslist.goodsCategory();
            },
            // 表单验证
            validate (prop, status, error) {
                if (status === false) {
                    this.$Message.error(error);
                }
            },
            // 添加自定义弹窗
            addCustomDialog (editorId) {
                window.UE.registerUI('test-dialog', function (editor, uiName) {
                    // 创建 dialog
                    let dialog = new window.UE.ui.Dialog({
                        // 指定弹出层中页面的路径，这里只能支持页面，路径参考常见问题 2
                        iframeUrl: '/admin/widget.images/index.html?fodder=dialog',
                        // 需要指定当前的编辑器实例
                        editor: editor,
                        // 指定 dialog 的名字
                        name: uiName,
                        // dialog 的标题
                        title: '上传图片',
                        // 指定 dialog 的外围样式
                        cssRules: 'width:1200px;height:500px;padding:20px;'
                    });
                    this.dialog = dialog;
                    var btn = new window.UE.ui.Button({
                        name: 'dialog-button',
                        title: '上传图片',
                        cssRules: `background-image: url(../../../assets/images/icons.png);background-position: -726px -77px;`,
                        onclick: function () {
                            // 渲染dialog
                            dialog.render();
                            dialog.open();
                        }
                    });
                    return btn;
                }, 37);
            }
        }
    }
</script>

<style scoped lang="stylus">
    .grey
        color #999;
    .maxW /deep/.ivu-select-dropdown{
        max-width 600px;
    }
    .ivu-table-wrapper
        border-left: 1px solid #dcdee2;
        border-top: 1px solid #dcdee2;
    .tabBox_img
        width 50px;
        height 50px;
    .tabBox_img img
        width 100%;
        height 100%;
    .priceBox
        width 100%
    .form
        .picBox
            display: inline-block;
            cursor: pointer;
        .pictrue
            width:60px;
            height:60px;
            border:1px dotted rgba(0,0,0,0.1);
            margin-right:15px;
            display: inline-block;
            position: relative;
            cursor: pointer;
            img
                width 100%
                height 100%
            .btndel
                position: absolute;
                z-index: 9;
                width :20px !important;
                height: 20px !important;
                left: 46px;
                top: -4px;
        .upLoad {
            width: 58px;
            height: 58px;
            line-height: 58px;
            border: 1px dotted rgba(0, 0, 0, 0.1);
            border-radius: 4px;
            background: rgba(0, 0, 0, 0.02);
            cursor: pointer;
        }

</style>

