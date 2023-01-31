<template>
    <div>
        <div class="i-layout-page-header">
          <div class="i-layout-page-header">
            <span class="ivu-page-header-title">{{$route.meta.title}}</span>
          </div>
        </div>
        <Card :bordered="false" dis-hover class="ivu-mt">
            <Form ref="authFrom" :model="authFrom"  :label-width="labelWidth" :label-position="labelPosition" @submit.native.prevent>
                <Row type="flex" :gutter="24">
                    <Col v-bind="grid">
                        <FormItem label="状态："  label-for="status1">
                            <Select v-model="authFrom.auth_type" placeholder="请选择" clearable element-id="status1" @on-change="userSearchs">
                                <Option value="0">未认证</Option>
                                <Option value="1">审核中</Option>
								<Option value="-1">审核拒绝</Option>
                            </Select>
                        </FormItem>
                    </Col>
                    <Col v-bind="grid">
                        <FormItem label="关键字："  label-for="title">
                            <Input search enter-button  v-model="authFrom.keywords"  placeholder="请输入关键字" @on-search="userSearchs"/>
                        </FormItem>
                    </Col>
                </Row>
            </Form>
            <Table :columns="columns1" :data="authLists" ref="table" class="mt25"
                   :loading="loading" highlight-row
                   no-userFrom-text="暂无数据"
                   no-filtered-userFrom-text="暂无筛选结果">
                <template slot-scope="{ row, index }" slot="sfz_photo">
                    <div class="tabBox_img" v-viewer>
                        <img v-lazy="row.sfz_photo">
                    </div>
                </template>
				<template slot-scope="{ row, index }" slot="sfz_gh">
				    <div class="tabBox_img" v-viewer>
				        <img v-lazy="row.sfz_gh">
				    </div>
				</template>
				<template slot-scope="{ row, index }" slot="yyzz_img">
				    <div class="tabBox_img" v-viewer>
				        <img v-lazy="row.yyzz_img">
				    </div>
				</template>
                <template slot-scope="{ row, index }" slot="auth_types">
					<span v-if="row.auth_type==-1" style="color: red;">审核拒绝</span>
					<span v-if="row.auth_type==0">未认证</span>
					<span v-if="row.auth_type==1">审核中</span>
					<span v-if="row.auth_type>1" style="color: green;">认证成功</span>
                </template>
                <template slot-scope="{ row, index }" slot="action">
                    <a @click="edit(row)" v-if="row.auth_type<=1">审核</a>
                </template>
            </Table>
            <div class="acea-row row-right page">
                <Page :total="total" :current="authFrom.page" show-elevator show-total @on-change="pageChange"
                      :page-size="authFrom.limit"/>
            </div>
        </Card>
		<Modal v-model="modals" scrollable  closable title="审核"  :z-index="1" width="700">
			<div class="msg_modals">
				<div style="font-size: 16px;margin: 10px 0;">审核说明：</div>
				<Input v-model="sh_msg" type="textarea" :rows="3" placeholder="请输入审核说明" />    
			</div>
		   <template #footer>
			   <Button type="error" @click="auth(-1)">拒绝</Button>
			   <Button type="success" @click="auth(2)">通过</Button>
		   </template>
		</Modal>
    </div>
</template>
<script>
    import { mapState } from 'vuex'
    import { authListApi, authSave } from '@/api/user'
    export default {
        name: 'user_level',
        data () {
            return {
				modals: false,
                grid: {
                    xl: 7,
                    lg: 7,
                    md: 12,
                    sm: 24,
                    xs: 24
                },
                loading: false,
                columns1: [
                    {
                        title: 'ID',
                        key: 'id',
                        width: 80
                    },
                    
                    {
                        title: '法人姓名',
                        key: 'name',
                        minWidth: 120
                    },
                    {
                        title: '法人手机号',
                        key: 'phone',
                        minWidth: 100
                    },
                    {
                        title: '企业名称',
                        key: 'company',
                        minWidth: 100
                    },
					{
					    title: '身份证正面',
					    slot: 'sfz_photo',
					    minWidth: 100
					},
					{
					    title: '身份证反面',
					    slot: 'sfz_gh',
					    minWidth: 100
					},
					{
					    title: '营业执照',
					    slot: 'yyzz_img',
					    minWidth: 100
					},
                    {
                        title: '认证状态',
                        slot: 'auth_types',
                        minWidth: 120
                    },
                    {
                        title: '审核说明',
                        key: 'sh_msg',
                        minWidth: 120
                    },
                    {
                        title: '操作',
                        slot: 'action',
                        fixed: 'right',
                        minWidth: 120
                    }
                ],
                authFrom: {
                    auth_type: '',
                    keywords: '',
                    page: 1,
                    limit: 15
                },
                authLists: [],
                total: 0,
                FromData: null,
                imgName: '',
                authId: 0,
                num: 0,
				sh_msg:''
            }
        },
        created () {
            this.getList()
        },
        computed: {
            ...mapState('media', [
                'isMobile'
            ]),
            labelWidth () {
                return this.isMobile ? undefined : 75
            },
            labelPosition () {
                return this.isMobile ? 'top' : 'right'
            }
        },
        methods: {
            // 列表
            getList () {
                this.loading = true
                this.authFrom.auth_type = this.authFrom.auth_type || ''
                authListApi(this.authFrom).then(async res => {
                    let data = res.data
                    this.authLists = data.list
                    this.total = res.data.count
                    this.loading = false
                }).catch(res => {
                    this.loading = false
                    this.$Message.error(res.msg)
                })
            },
            pageChange (index) {
                this.authFrom.page = index
                this.getList()
            },
            
            // 审核
            edit (row) {
                this.authId = row.id
                this.modals = true
            },
			auth(type){
				let data = {
					id:this.authId,
					sh_msg:this.sh_msg,
					auth_type:type
				}
				authSave(data).then(async res => {
				    this.$Message.success(res.msg)
					this.modals = false
					this.getList()
					this.sh_msg = ''
				}).catch(res => {
				    this.$Message.error(res.msg)
				})
			},
            // 表格搜索
            userSearchs () {
                this.authFrom.page = 1
                this.getList()
            },
        }
    }
</script>

<style scoped lang="stylus">
    .tabBox_img
        width 36px
        height 36px
        border-radius:4px
        cursor pointer
        img
            width 100%
            height 100%
</style>
