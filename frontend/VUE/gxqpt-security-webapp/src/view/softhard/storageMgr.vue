<!-- 存储管理 -->
<template>
	<Layout>
		<Content>
			<Breadcrumb>
				<BreadcrumbItem>{{ title }}</BreadcrumbItem>
			</Breadcrumb>
			<Card>
				<Form ref="formValidate" inline :label-width="110" :model="serverData">
					<FormItem label="服务器实例ID：">
             <Input type="text" v-model="serverData.serverId" style="width:140px"></Input>
					</FormItem>
          <FormItem label="服务器ip：">
             <Input type="text" v-model="serverData.ip" style="width:140px"></Input>
					</FormItem>
          <FormItem label="管理员：">
             <Input type="text" v-model="serverData.manageUname" style="width:140px"></Input>
					</FormItem>
					<FormItem label="内存大小：">
						<Select class="queryItem" clearable v-model="serverData.memorySize">
							<Option value="2">2GB</Option>
							<Option value="4">4GB</Option>
							<Option value="8">8GB</Option>
							<Option value="16">16GB</Option>
							<Option value="32">32GB</Option>
							<Option value="64">64GB</Option>
						</Select>
					</FormItem>
          <FormItem label="磁盘大小：">
             <Input type="text" v-model="serverData.diskSize" style="width:140px"></Input>
					</FormItem>
					<FormItem :label-width="20">
						<Button type="primary" @click="search">查询</Button>
					</FormItem>
				</Form>
				<hy-table ref="selection" :data="data" :columns="columns" :current="pageOption.current" :total="pageOption.total" :page-size="pageOption.pageSize" @on-change="pageChange" @on-page-size-change="changePageSize" show-sizer show-elevator/>
			</Card>
		</Content>
	</Layout>
</template>

<script>
	import api from '@/api/axiosApi'
	import softhardApiList from '@/api/softhardApiList'
	import { mapState } from 'vuex'
	export default {
		data() {
			return {
				title:this.$store.state.title,
				columns: [{
						type: 'index',
						title: '序号',
						width: 60,
						align: 'center'
					},
					{
						title: '服务器ip',
						key: 'ip'
          },
          {
						title: '服务器实例ID',
						key: 'serverId'
					},
					{
						title: '管理员',
						key: 'manageUname'
          },
					{
						title: '内存大小',
						key: 'memorySize'
          },
          {
						title: '内存使用率',
						key: 'memoryPercent'
					},
					{
						title: '磁盘大小',
						key: 'diskSize'
					},
					{
						title: '磁盘使用率',
						key: 'diskPercent'
					},
					{
						title: '创建时间',
						width: 220,
						align: 'center',
						key: 'createTime'
					},
					{
						title: '操作',
						key: 'act',
						width: 160,
						render: (h, params) => {
							const capacity = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_storage_management_kr')?'inline-block':'none'
								},
								on: {
									click: () => {
										this.gotoCtrl(2,params.row);										
									}
								}
							}, '扩容');
							const free = h('Button', {
								props: {
									type: 'primary',
									size: 'small'
								},
								style: {
									display:this.checkButton('hardware_storage_management_sf')?'inline-block':'none'
								},
								on: {
									click: () => {
										console.log(params.row)
										this.releaseServer(params.row.id);
										// this.gotoCtrl(2,params.row);										
									}
								}
							},"释放");
							return h('div', [capacity, free]);
						}
					}
				],
				data: [],
        serverData: { //查询参数
          serverId:"",
          serverName:"",
					admin: "",
					memorySize: "",
					diskSize: "",
				},
				pageOption: { //分页参数
					current: 1,
					total: 0,
					pageSize: 10
				},
				sysVal: "",
			}
		},
		mounted() {
			this.getMemoryResManage(); //查询服务分配管理分页
			console.log(this.$store.state);
		},
		methods: {
			handleChange(value, selecteddata) {
				if(selecteddata.length>0){
					let val = selecteddata.map(o => o.value).reverse();
					this.serverData.system = val[0];
				}else{
					this.serverData.system = "";
				}
			},
			pageChange(num) { //页码改变的回调
				this.pageOption.current = num;
				this.getMemoryResManage();
			},
			changePageSize(num) { //切换每页条数时的回调
				this.pageOption.pageSize = num;
				this.getMemoryResManage();
			},
			releaseServer(id){//释放服务器数据
				const vm = this
				vm.$Modal.confirm({
					title: '释放确认',
					content: '确认释放此条信息吗？',
					onOk: () => {
						api(softhardApiList.releaseServer, {
							id: id
						}).then(res => {
							if (res.status == 200 && res.data.data=="SUCCESS") {
								vm.$Message.success('释放成功！')
								vm.search()
							}
						})
					}
				})
			},
			getMemoryResManage() { // 查询服务分配管理分页
				this.data = [];
				var formData = {
					"data": { ...this.serverData
					}, //Object.assign({}, this.searchCondition),
					"pageNo": this.pageOption.current,
					"pageSize": this.pageOption.pageSize
				};
				api(softhardApiList.getMemoryResManage, formData).then((res) => {
					if(res.status == 200 && res.data.data) {
						this.data = res.data.data.list;
						if(this.data.length>0){
							this.pageOption.pageSize = res.data.data.pageSize;
							this.pageOption.total = res.data.data.total;
							this.pageOption.current = res.data.data.pageNum;
							this.data.forEach(item=>{
								item.createTime = item.createTime.replace("-","年");
								item.createTime = item.createTime.replace("-","月");
								item.createTime = item.createTime.replace(" ","日 ");
							})
						};
					}
				}, (err) => {
					//dong something...
				})
			},
			search() {
				this.pageOption.current=1;
				console.log(this.serverData);
				this.getMemoryResManage();
			},	
			// 操作
			gotoCtrl(idx,data) {
				// 配置变更
				if(idx === 2) {
					this.$router.push({
						name: 'modify',
						params: {id: idx,departData:data}
					})
				} else { // 详情
					this.$router.push({
						path: '/department/detail'
					})
				}
			},
			checkButton(code){
				if(this.authButton.indexOf(code) >= 0){
					return true;
				}else{
					return false;
				}
			}
		},
		computed: {
			...mapState([
				'authButton'
			])
		},
	}
</script>

<style lang='less' scoped>
	.ivu-btn-small {
		margin: 0 3px;
	}
	.handle{
		color:red;
		cursor: pointer;
	}
	.queryItem {
		width: 200px;
	}
	.access-list {
		span.handle {
			margin: 0 5px;
			display: inline-block;
			cursor: pointer;
			&:hover {
				color: #57a3f3;
			}
		}
	}
</style>
<style lang="less">
.ivu-message{
  z-index: 9999;
}
</style>