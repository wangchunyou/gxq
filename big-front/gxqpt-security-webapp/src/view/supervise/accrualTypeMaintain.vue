<template>
  <Layout>
    <Content>
      <Breadcrumb>
        <BreadcrumbItem>基础信息维护</BreadcrumbItem>
        <BreadcrumbItem>权责类型维护</BreadcrumbItem>
      </Breadcrumb>
      <Card style="min-height: 600px;">
        <Row>
          <Col span="6">
            <Tree :data="ztreeData" :render="renderContent" id="serviceList"></Tree>
          </Col>
          <Col span="16">
            <template>
              <Form :model="currentNodeInfo" :label-width="100">
                <FormItem label="分类名称：">
                  <Row>
                    <Col span="24">
                        <Input :value="currentNodeInfo.name" disabled></Input>
                    </Col>
                  </Row>
                </FormItem>
                <FormItem label="分类编码：">
                  <Row>
                    <Col span="24">
                        <Input :value="currentNodeInfo.code" disabled></Input>
                    </Col>
                  </Row>
                </FormItem>
                <FormItem label="分类描述：">
                  <Row>
                    <Col span="24">
                        <Input type="textarea" :value="currentNodeInfo.desc" :autosize="{minRows: 5, maxRows: 25}" disabled></Input>
                    </Col>
                  </Row>
                </FormItem>
              </Form>
            </template>
          </Col>
        </Row>
      </Card>
      <Modal v-model="modal" :title="actTitle" :mask-closable="false">
        <Form ref="modalForm" inline :label-width="90" :rules="ruleValidate" :model="modalFormData">
          <!-- 不是添加最后一级或者编辑最后一级 -->
          <!-- <template v-if="notOperateLastLevel">
            <FormItem :label="inputLabel" prop="type">
              <Input type="text" v-model="modalFormData.type" style="width:150px;"></Input>
            </FormItem>
          </template> -->
          <template>
            <FormItem label="分类名称" prop="name">
              <Input type="text" v-model="modalFormData.name" style="width:200px;"></Input>
            </FormItem>
            <FormItem label="分类编码" prop="code">
              <Input type="text" v-model="modalFormData.code" style="width:200px;"></Input>
            </FormItem>
            <FormItem label="分类描述" style="width: 100%;" prop="desc">
              <Input
                type="textarea"
                v-model="modalFormData.desc"
                :rows="5"
                :autosize="{minRows: 5, maxRows: 5}">
              </Input>
            </FormItem>
          </template>
        </Form>
        <div slot="footer">
          <Button type="default" @click="modal = false" size="large">取消</Button>
          <Button type="primary" @click="handleSubmit" size="large">确定</Button>
        </div>
      </Modal>
      <Modal v-model="moveModal" title="移动节点" :mask-closable="false">
        <Tree :data="filterTreeData" ref="filterTree"></Tree>
        <div slot="footer">
          <Button type="default" @click="moveModal = false" size="large">取消</Button>
          <Button type="primary" @click="moveNode" size="large">确定</Button>
        </div>
      </Modal>
    </Content>
  </Layout>
</template>

<script>
import {mapState} from 'vuex'
import api from '@/api/axiosApi'
import {changeTreeData} from '@/assets/js/utils'
import superviseApiList from '@/api/superviseApiList'

// 操作类型
const actTypes = {
  edit: 'edit',
  create: 'create'
}

export default {
  data() {
    return {
      // 树的所有节点数组
      treeRoot: [],
      // 当前操作的节点对象，包含父节点的nodekey
      currentNode: {},
      actTypes,
      modal: false,
      // 当前操作的类型
      currentAct: 'edit',
      // 当前操作的节点的信息
      currentNodeInfo: {
        // 自生的id
        id: '',
        // 父节点的id，如果没有父节点就为-1
        parentId: ''
      },
      // 新增或者编辑时的表单
      modalFormData: {
        title: '',
        desc: '',
        code: ''
      },
      ruleValidate: {
        name: [{required: true, message: '不可为空', trigger: 'blur'}, {max: 64, message: '请不要输入长度超过64字的分类名称', trigger: 'blur' }],
        code: [{required: true, message: '不可为空', trigger: 'blur'}, {max: 64, message: '请不要输入长度超过64的分类编码', trigger: 'blur' }],
        desc: [{max: 500, message: '请不要输入超过500字的分类描述', trigger: 'blur' }],
      },
      ztreeData: [],
      filterTreeData: [],
      // 移动选择弹窗
      moveModal: false
    }
  },
  computed: {
    actTitle () {
      return this.currentAct === actTypes.edit ? '编辑分类' : '新增分类'
    },
    saveOrUpdateUrl() {
      return this.currentAct === actTypes.create ? 'saveTypeTree' : 'updateTypeTree'
    },
    inputLabel () {
      return ['', '分类名称', '分类编码', '分类描述'][this.currentNodeInfo.level + 1 || 0]
    },
    // 不是操作最后一级？
    notOperateLastLevel () {
      // 新建，但是level不是3，因此不是最后一级，因为只能添加子类
      if (this.currentAct === actTypes.create && this.currentNodeInfo.level !== 3) {
        return true
      }
      // 编辑，但是level不是4，因此不是编辑最后一级
      if (this.currentAct === actTypes.edit && this.currentNodeInfo.level !== 4) {
        return true
      }
      return false
    },
    ...mapState(['authButton'])
  },
  methods: {
    renderContent(h, { root, node, data }) {
      const vm = this
      const defaultBtnStyle = {
        marginRight: '8px'
      }
      const buttonProps = {
        type: 'ghost',
        size: 'small',
      }
      // 添加按钮
      const createBtn = h('Button', {
        attrs: {
          title: '添加'
        },
        props: {
          icon: 'plus',
          ...buttonProps
        },
        style: {
          color: 'green',
          ...defaultBtnStyle
        },
        on: {
          click: () => {
            this.treeRoot = root
            this.currentNode = node
            this.currentNodeInfo = data
            this.currentAct = actTypes.create
            this.openModal()
          }
        }
      })
      // 编辑按钮
      const editBtn = h('Button', {
        attrs: {
          title: '修改'
        },
        props: {
          icon: 'edit',
          ...buttonProps
        },
        style: {
          color: 'blue',
          ...defaultBtnStyle
        },
        on: {
          click: () => {
            this.treeRoot = root
            this.currentNode = node
            this.currentNodeInfo = data
            this.currentAct = actTypes.edit
            this.openModal(data.id)
          }
        }
      })
      // 删除按钮
      const deleteBtn = h('Button', {
        attrs: {
          title: '删除'
        },
        props: {
          icon: 'close',
          ...buttonProps
        },
        style: {
          color: 'red',
          ...defaultBtnStyle
        },
        on: {
          click: () => {
            this.treeRoot = root
            this.currentNode = node
            this.currentNodeInfo = data
            this.overdueById(data.id)
          }
        }
      })
      // 移动按钮
      const moveBtn = h('Button', {
        attrs: {
          title: '移动节点'
        },
        props: {
          icon: 'paper-airplane',
          ...buttonProps
        },
        style: {
          color: '#57C5F7',
          ...defaultBtnStyle
        },
        on: {
          click: () => {
            this.treeRoot = root
            this.currentNode = node
            this.currentNodeInfo = data
            this.moveItemById(data.id)
          }
        }
      })
      let title = data.title
			const btns = new Set()
			//按钮权限
      // if (vm.authButton.includes('ops_server_manage_add')) {
      //   btns.add(createBtn)
      // }
      // if (vm.authButton.includes('ops_server_manage_edit')) {
      //   btns.add(editBtn)
      // }
      // if (vm.authButton.includes('ops_server_manage_delete')) {
      //   btns.add(deleteBtn)
			// }
			if (data.level<4) {
			  btns.add(createBtn)
			}
			btns.add(editBtn)
      btns.add(deleteBtn)
      btns.add(moveBtn)
      // 如果有子节点，那就不能删除
      if (data.children && data.children.length > 0) {
        btns.delete(deleteBtn)
      }
      // 最后一级不能添加
      if (data.level === 4) {
        btns.delete(createBtn)
        title = data.title
      }
      // 根节点不能移动
      if (data.parentId === '-1') {
        btns.delete(moveBtn)
      }
      return h('span', {
        attrs: {
          class: 'doc-tree-title'
        }
      }, [
        h('span', {
          on: {
            click: () => {
							this.getTypeTreeById(data.id)
              // 点击文字也可以展开树结构
              if (data.disabled) {
                return
              }
              Vue.set(data, 'expand', !data.expand)
            }
          },
          attrs: {
            class: 'title'
          }
        }, title),
        h('span', {
          attrs: {
            class: 'act-btns-container'
          }
        }, [...btns])
      ])
		},
    // 移动节点
    moveItemById (id) {
      const vm = this
      const filterTreeData = JSON.parse(JSON.stringify(vm.ztreeData))
      for (let item of filterTreeData) {
        if (vm.filterTree(item)) {
          break;
        }
      }
      vm.filterTreeData = filterTreeData;
      vm.moveModal = true;
    },
    // 过滤掉不可选的节点，用于移动
    filterTree (data) {
      const vm = this
      if (data.id === vm.currentNodeInfo.id) {
        data.disabled = true;
        return true;
      } else if (data.children.length > 0) {
        for (let item of data.children) {
          if (vm.filterTree(item)) {
            return false;
          }
        }
      }
      return false;
    },
    moveNode () {
      const vm = this;
      const node = vm.$refs.filterTree.getSelectedNodes();
      if (node.length === 0) {
        vm.$Message.error('请选择一个节点作为父节点');
        return;
      }
      if (node[0].level === 4) {
        vm.$Message.error('该节点为底层节点，不能作为父节点，请重新选择');
        return;
      }
      vm.$Modal.confirm({
        title: '节点移动确认',
        content: `确定将节点“${vm.currentNodeInfo.title}”移动到节点“${node[0].title}”下吗？`,
        onOk: () => {
          vm.updateLocation(node[0].id);
        }
      })
    },
    async updateLocation (locateId) {
      const vm = this
      const res = await api(superviseApiList.updateLocation, {
        currentId: vm.currentNodeInfo.id,
        locateId,
        type: 'inner'
      })
      if (res.data.errcode === 0) {
        vm.getTreeData();
        vm.$Message.success('移动成功');
      } else {
        vm.$Message.error(res.data.errmsg);
      }
      vm.moveModal = false;
    },
    // 根据id查询权责类型信息
		getTypeTreeById(id) {
      const vm = this
      api(superviseApiList.getTypeTreeById, {id: id,}).then(res => {
        if (res.data.errcode === 0) {
					this.currentNodeInfo = res.data.data;
					this.modalFormData = res.data.data;
        }
      }, error => {console.log(error)})
    },
    // 获取树结构
    getTreeData(isAct, node) {
      const vm = this
      api(superviseApiList.findPowerSuperviseTypeTree, {
        id: '',
        timestamp: Date.now()
      }).then(res => {
        // if (res.data.errcode === 0) {
				// 		const rootNode = res.data.data;
				// 		rootNode.forEach(item => {
				// 			item.expand = true;
				// 		});
				// 		this.ztreeData = JSON.parse(JSON.stringify(rootNode).replace(/name/g, 'title'));
        // }
        if (res.data.errcode === 0) {
          res.data.data = vm.builderTree(res.data.data, 0)
          res.data.data[0].expand = true
          // 操作--新增
          if (isAct) {
            const opts = {
              root: vm.treeRoot,
              node: vm.currentNode,
              data: vm.currentNodeInfo,
              nodeInfo: node
            }
            // 新增子类
            /*if (vm.modalFormData.type === '1') {
              opts.type = 'createChild'
            } else {
              opts.type = 'create'
            }*/
            opts.type = 'createChild'
            changeTreeData(opts)
          } else {
            if (node) {
              vm.treeRoot[vm.currentNode.nodeKey].node.title = node.name
            } else {
              const ztreeData = JSON.parse(JSON.stringify(res.data.data).replace(/name/g, 'title'))
              ztreeData[0].disabled = true;
              vm.ztreeData = ztreeData;
            }
          }
        } else {
          vm.$Message.error(res.data.errmsg) 
        }
      }, error => {console.log(error)})
    },
    builderTree(r, num) {
      let _this = this;
      if(!r || r.length == 0) {
        return;
      }
      let arrayFirst = [];
      r.forEach(function(value, index) {
        let obj = value;
        obj.level = num;
        if(value.children.length > 0) {
          obj.children = _this.builderTree(value.children, num + 1);
        };
        arrayFirst.push(obj);
        return;
      });
      return arrayFirst;
    },
    // 保存或者更新节点信息
    saveTypeTree() {
      const vm = this
      const {parentId, id, level} = {...vm.currentNodeInfo}
      let data = {
        orderNum: 1,
        ...vm.modalFormData
      }
      // 编辑
      if (vm.currentAct === actTypes.edit) {
        data.id = id
      } else {
        // 新增子类
        data.parentId = id
        data.id = 0
        data.level = level + 1
      }
      api(superviseApiList[vm.saveOrUpdateUrl], data)
      .then(res => {
        if (res.data.errcode === 0) {
          if(vm.currentAct === 'edit'){
            changeTreeData({
              root: vm.treeRoot,
              node: vm.currentNode,
              data: vm.currentNodeInfo,
              nodeInfo: vm.modalFormData,
              type: 'edit'
            })
            vm.getTreeData(false, res.data.data)
            vm.$Message.success('编辑成功！')  
          }else{
            vm.getTreeData(true, res.data.data)
            vm.$Message.success('添加成功！') 
          }
          vm.modal = false
        } else {
          vm.$Message.error(res.data.errmsg) 
        }
      }, error => {console.log(error)})
    },
    handleSubmit () {
      const vm = this
      vm.$refs.modalForm.validate(valid => {
        if (valid) {
          vm.saveTypeTree()
        } else {
          this.$Message.error('表单验证失败!')
        }
      })
    },
    // 删除节点信息
    overdueById(id) {
      const vm = this
      vm.$Modal.confirm({
        title: '删除确认',
        content: '确认删除吗？',
        onOk: () => {
          api(superviseApiList.overdueById, {id: id}).then(res => {
            if (res.data.errcode === 0) {
              vm.$Message.success('删除成功！')
              changeTreeData({
                root: vm.treeRoot,
                node: vm.currentNode,
                data: vm.currentNodeInfo,
                type: 'delete'
              })
              vm.modal = false
            }
          }, error => {console.log(error)})
        }
      })
    },
    // 打开表单，并且初始化
    openModal(id) {
      this.modal = true
			this.$refs.modalForm.resetFields();
      // 编辑
      if (this.currentAct === actTypes.edit) {
				this.getTypeTreeById(id);
        // this.modalFormData = {
        //   // type: this.currentNodeInfo.title,
        //   name: this.currentNodeInfo.name,
        //   desc: this.currentNodeInfo.desc,
        //   code: this.currentNodeInfo.code
        // }
      }
    }
  },
  mounted() {
    this.getTreeData()
  }
}
</script>

<style type="text/css" scoped>
.w168 {
  width: 168px;
}
</style>

<style lang="less">
#serviceList{
  .doc-tree-title{
    display: inline-block;
    width: 100%;
    vertical-align: middle;
    .title{
      display: inline-block;
      height: 24px;
      float: left;
      line-height: 24px;
      cursor: pointer;
    }
    .act-btns-container{
      display: none;
      margin-left: 10px;
    }
    &:hover{
      .act-btns-container{
        display: inline-block;
      }
    }
  }
}

</style>
