<template>
	<div id="examine">
		<Form ref="docData" :model="docData" :rules="ruleValidate" :label-width="130">
			<div v-if="docData.isTrue=='1'">
				<Row>
					<Col span="12">
					<FormItem label="故障类别：">
						<Input v-model="docData.faultType" type="text" disabled></Input>
					</FormItem>
					</Col>
					<Col span="11">
					<FormItem label="故障级别：">
						<Select v-model="docData.faultLevel" disabled>
							<Option value="0">全部</Option>
							<Option value="1">一级</Option>
							<Option value="2">二级</Option>
							<Option value="3">三级</Option>
						</Select>
					</FormItem>
					</Col>
				</Row>
				<FormItem label="故障简述：">
					<Row>
						<Col span="23">
						<Input v-model="docData.faultInfo" type="textarea" :rows="5" disabled></Input>
						</Col>
					</Row>
				</FormItem>
				<FormItem label="处理时间：" prop="dealTime">
					<Row>
						<Col span="12">
						<DatePicker @on-change="changeTime" v-model="docData.dealTime" type="datetime"> </DatePicker>
						</Col>
					</Row>
				</FormItem>
				<FormItem label="处理流程：" prop="dealProcess">
					<Row>
						<Col span="23">
						<Input v-model="docData.dealProcess" type="textarea" :rows="5" :maxlength="1000"></Input>
						</Col>
					</Row>
				</FormItem>
				<FormItem label="结果：" prop="result">
					<Row>
						<Col span="23">
						<Input v-model="docData.result" type="textarea" :rows="5" :maxlength="1000"></Input>
						</Col>
					</Row>
				</FormItem>
				<FormItem label="建议：" prop="advice">
					<Row>
						<Col span="23">
						<Input v-model="docData.advice" type="textarea" :rows="5" :maxlength="1000"></Input>
						</Col>
					</Row>
				</FormItem>
				<FormItem label="附件：" style="margin-top: 40px;">
					<hy-upload
						ref="evalReport"
						multiple
						:on-success="setEvalReport"
						:on-remove="removeEvalReport"
						:before-upload="evalBeforeUpload"
						:format="['xls','xlsx','doc', 'docx','pdf']"
						:defaultFileList="docData.fileList">
					</hy-upload>
				</FormItem>
				<Row>
					<Col span="12">
					<FormItem label="是否加入知识库：">
						<Select v-model="docData.isJoinKnowledge">
							<Option value="1">是</Option>
							<Option value="2">否</Option>
						</Select>
					</FormItem>
					</Col>
				</Row>
			</div>
			<FormItem label="理由：" prop="docDescribe" v-if="docData.isTrue=='2'">
				<Row>
					<Col span="24">
					<Input v-model="docData.dealOpinion" type="textarea" :rows="5"></Input>
					</Col>
				</Row>
			</FormItem>
		</Form>
	</div>
</template>

<script>
	import hyUpload from '@/components/hengyun/hyUpload'
	export default {
		props: ['dealReport'],
		data() {
			return {
				docData: {
					isTrue: "1",
					isJoinKnowledge: "1",
					faultType: '',
					faultLevel: '',
					faultInfo: '',
					id: '',
					fileList: [],
					dealProcess: '',
					result: '',
					advice: '',
					changeTime: ''
				},
				ruleValidate: {
					dealTime: [{
						required: true,
						message: '该项为必填项，请填写相应数据！',
						trigger: 'change'
					}],
					dealProcess: [{
						required: true,
						message: '该项为必填项，请填写相应数据！',
						trigger: 'blur'
					}],
					result: [{
						required: true,
						message: '该项为必填项，请填写相应数据！',
						trigger: 'blur'
					}],
					advice: [{
						required: true,
						message: '该项为必填项，请填写相应数据！',
						trigger: 'blur'
					}],
				},
			}
		},
		components:{
			hyUpload
		},
		methods: {
			handleSubmit(name) {
				this.$refs[name].validate((valid) => {
					if(valid) {
						this.$emit('examine', this.docData);
					}
				})
			},
			getDetail(data) {
				this.docData.faultType = data.faultType;
				this.docData.faultLevel = data.faultLevel;
				this.docData.faultInfo = data.faultInfo;
				this.docData.isTrue = data.isTrue;
				this.docData.id = data.id;
			},
			setEvalReport(response, file, fileList) {
				let _this = this;
				if(this.$route.params.id != 3) {
					this.docData.fileList = [];
				}
				fileList.forEach(item => {
					let obj = {};
					if(item.response) {
						obj.bussId = _this.docData.id;
						obj.bussType = 'fault_report';
						obj.fileName = item.response.data.list[0].submittedFileName;
						obj.fileSize = item.response.data.list[0].size;
						obj.fileType = item.response.data.list[0].ext;
						obj.fileUrl = item.response.data.list[0].url;
						obj.name = item.response.data.list[0].submittedFileName;
						this.docData.fileList.push(obj);
					}
				});
			},
			removeEvalReport(file, fileList) {
				this.docData.fileList = [];
				this.docData.fileList = fileList;
			},
			evalBeforeUpload() { //文件上传前清空
				// this.$refs.evalReport.$children[0].clearFiles();
			},
			changeTime(val) {
				if(val) {
					this.docData.dealTime = val;
				} else {
					this.docData.dealTime = "";
				}
			},
		},
		watch: {
			'dealReport' () {
				this.$refs.docData.resetFields()
				this.docData.fileList = []
				this.docData.isJoinKnowledge = '1'
			}
		}
	}
</script>

<style>
	#reportDeal .ivu-modal-body {
		min-height: 240px;
		max-height: 600px;
		overflow-y: auto;
	}
</style>