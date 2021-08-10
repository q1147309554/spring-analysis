
<template>
  <div class="userManagement-wrap">
    <div class="userManagement-body">
      <!-- 操作界面 编辑等 -->
      <div class="userm-a">
        <el-dialog
          :title="textMap[dialogStatus]"
          :visible.sync="dialogFormVisible"
          :append-to-body="true"
          class="userm-dialog"
        >
          <div class="user-all">
            <div class="usergroup">
              <div>
                <div class="tree-transfer">
                  <!-- 穿梭框左侧 -->
                  <div class="tree-transfer-left">
                    <el-tree
                      ref="treeLeft"
                      show-checkbox
                      node-key="id"
                      lazy
                      :load="loadNode"
                      :indeterminate="true"
                      @check-change="handleCheckChange"
                      :props="defaultProps"
                    >
                    </el-tree>
                  </div>
                  <div class="tree-transfer-middle">
                    <el-button
                      circle
                      type="info"
                      icon="el-icon-arrow-left"
                      @click="remove"
                    ></el-button>
                    <el-button
                      circle
                      type="info"
                      icon="el-icon-arrow-right"
                      @click="add"
                    ></el-button>
                  </div>
                  <!-- 穿梭框右侧 -->
                  <div class="tree-transfer-right">
                    <el-tree
                      ref="treeRight"
                      :data="rightData"
                      show-checkbox
                      node-key="id"
                      :props="defaultProps"
                    >
                    </el-tree>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </el-dialog>
      </div>
    </div>
  </div>
</template>

<script>
// import treeTransfer from 'el-tree-transfer' // 引入
const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})
export default {
  name: 'userGroupManagement',
  components: { },
  filters: {
    statusFilter (status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter (type) {
      return calendarTypeKeyValue[type]
    }
  },
  data () {
    return {
      tempData: [],
      tempList: [],
      mode: 'transfer',
      disabled: false,
      leftData: [],
      rightData: [],
      defaultProps: {
        children: 'children',
        label: 'displayName',
        isLeaf: 'leaf'
      },
      datas: [],
      userwrap: [], // 穿梭框数据
      userwrapValue: [],
      rolewrap: [],
      rolewrapValue: [],
      distatus: false,
      entityTotalProps: 1,
      entityCurrentPageProps: 1,
      entityPagesizeProps: 0,
      textMap: {
        update: '编辑',
        create: '新增'
      },
      calendarTypeOptions,
      temp: {
        id: undefined,
        name: '',
        showName: '',
        describe: '',
        createtime: new Date()
      },
      totalPages: '',
      
      dialogStatus: '',
      dialogFormVisible: false,
      multipleSelection: [],
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '+id'
      },
      importanceOptions: [1, 2, 3],
      sortOptions: [
        { label: 'ID Ascending', key: '+id' },
        { label: 'ID Descending', key: '-id' }
      ],
      showReviewer: false,
      downloadLoading: false
    }
  },
  created () {
    this.handleCreate()
  },
  methods: {
    handleData (list) {
      const _repeatArr = this.tempData.concat([], list)
      return this.handleRepeatData(_repeatArr) // _repeatArr 处理出来 存在重复数据 此处需要将此去重
    },
    handleRepeatData (_repeatArr) {
      const _arr = []
      _repeatArr.map(item => {
        if (!item.leaf) return
        const res = _arr.every(itm => !(item.id === itm.id))
        if (!res) return
        _arr.push(item)
      })
      return _arr
    },
    add () {
      // 左 往 右
      // 清除右侧
      this.rightData = []
      this.leftData = this.$refs.treeLeft.getCheckedNodes()
      console.log('左侧选中', this.leftData, this.tempData)
      const res = this.handleData(this.leftData)
      this.rightData.push(...res)
    },
    remove () {
      // 删除 右侧
      const _list = this.$refs.treeRight.getCheckedNodes()
      // 清除右侧数据
      _list.map(item => {
        const index = this.rightData.findIndex(itm => {
          return item.id === itm.id
        })
        if (index >= 0) {
          this.rightData.splice(index, 1)
        }
      })
      // 清除左侧数据
      const deleteData = _list.map(item => item.id)
      const leftChecked = this.$refs.treeLeft.getCheckedKeys()
      const resArr = []
      leftChecked.map(item => {
        if (String(item).indexOf('-') > 0 && !new Set(deleteData).has(item)) {
          resArr.push(item)
        }
      })
      this.$refs.treeLeft.setCheckedKeys(resArr)
    },
    loadNode (node, resolve) {
      // 左侧树懒加载 真实情况为接口数据加载
      if (node.level === 0) {
        return resolve(this.datas)
      }
      if (node.level === 1) {
        return resolve(node.data.children)
      }
      if (node.level > 1) {
      // 判断一个对象是否拥有该属性的时候 如 o = {name: 'admin'}不要使用o.hasOwnProperty("name")  使用 Object.prototype.hasOwnProerty.call(o, "name") 会导致意外行为（恶意修改hasOwnProperty的值） 或拒绝服务安全漏洞
        if (Object.prototype.hasOwnProperty.call(node.data, 'children')) {
          return resolve(node.data.children)
        } else {
          setTimeout(() => {
            const data = {
              code: 200,
              msg: null,
              data: [
                {
                  realName: '李易峰',
                  company: '一处一科',
                  id: node.data.id + 100,
                  userName: '李易峰520 - ' + node.data.id
                },
                {
                  realName: '李易峰111111',
                  company: '一处一科',
                  id: node.data.id + 1001,
                  userName: '李易峰521 - ' + node.data.id
                }
              ]
            }
            //   this.$get(`/user/getUserByCompany?company=${node.data.name}`).then(
            // res => {
            if (data.code === 200) {
              const d = []
              data.data.forEach(itm => {
                const e = {}

                e.id = itm.id
                e.displayName = itm.userName
                e.name = itm.realName
                e.leaf = true
                d.push(e)
              })
              resolve(d)
            }
          }, 1000)
          // }
        //   )
        }
      }
    },
    getChilds (data) {
      if (data.id === 1) return
      if (data.children) {
        this.getChilds(data.children)
      } else {
        this.tempList.push(data)
      }
    },
    handleCheckChange (data, checked, indeterminate) {
      this.getChilds(data)
      const _list = this.tempList.flat(Infinity)
      console.log(_list)
      this.deepData(_list, checked)
    },
    deepData (_list, checked) {
      const _arr = []
      _list.map(async item => {
        const _obj = await this.simulationData(item)
        item.checked = checked
        this.$set(item, 'checked', checked)
        _arr.push(_obj)
        this.tempData = this.tempData.concat([], _arr.flat())
        // if (checked && item.id === itm.id) {
        //   _arr.push(_obj)
        //   this.tempData = this.tempData.concat([], _arr.flat())
        // }
        // 删除的功能 - 尚未写好
      })
    },
    async simulationData (item) {
      return new Promise(resolve => {
        const res = {
          code: 200,
          msg: null,
          data: [
            {
              realName: '李易峰',
              company: '一处一科',
              id: item.id + 100,
              userName: '李易峰520 - ' + item.id
            },
            {
              realName: '李易峰111111',
              company: '一处一科',
              id: item.id + 1001,
              userName: '李易峰521 - ' + item.id
            }
          ]
        }
        // const res = await this.$get(`/user/getUserByCompany?company=${item.name}`)
        if (res.code === 200) {
          const d = []
          res.data.forEach(itm => {
            const e = {}

            e.id = itm.id
            e.displayName = itm.userName
            e.name = itm.realName
            e.leaf = true
            d.push(e)
          })
          // return d
          resolve(d)
        } else {
          // return []
          resolve([])
        }
      })
    },
    // 获取力量模型数据
    async getPowerData () {
      const data = {
        code: 200,
        msg: null,
        data: [{
          children: [{ displayName: '一中心', name: '一中心', id: 2, parentId: 1 },
            { displayName: '二中心', name: '二中心', id: 3, parentId: 1 }, { displayName: '三中心', name: '三中心', id: 4, parentId: 1 },
            {
              children: [{ displayName: '一处一科', name: '一处一科', id: 6, parentId: 5 }, {
                displayName: '一处二科',
                name: '一处二科',
                id: 7,
                parentId: 5
              }, { displayName: '一处三科', name: '一处三科', id: 8, parentId: 5 }, {
                displayName: '一处四科',
                name: '一处四科',
                id: 9,
                parentId: 5
              }, { displayName: '一处五科', name: '一处五科', id: 10, parentId: 5 }],
              displayName: '一处',
              name: '一处',
              id: 5,
              parentId: 1
            }, {
              children: [{ displayName: '二处一科', name: '二处一科', id: 12, parentId: 11 }, {
                displayName: '二处二科',
                name: '二处二科',
                id: 13,
                parentId: 11
              }, { displayName: '二处三科', name: '二处三科', id: 14, parentId: 11 }, {
                displayName: '二处四科',
                name: '二处四科',
                id: 15,
                parentId: 11
              }, { displayName: '二处五科', name: '二处五科', id: 16, parentId: 11 }],
              displayName: '二处',
              name: '二处',
              id: 11,
              parentId: 1
            }, {
              children: [{ displayName: '三处一科', name: '三处一科', id: 18, parentId: 17 }, { displayName: '三处二科', name: '三处二科', id: 19, parentId: 17 }],
              displayName: '三处',
              name: '三处',
              id: 17,
              parentId: 1
            }, {
              children: [{ displayName: '四处一科', name: '四处一科', id: 21, parentId: 20 }, { displayName: '四处二科', name: '四处二科', id: 22, parentId: 20 },
                { displayName: '四处三科', name: '四处三科', id: 23, parentId: 20 }, { displayName: '四处四科', name: '四处四科', id: 24, parentId: 20 }, { displayName: '四处五科', name: '四处五科', id: 25, parentId: 20 }],
              displayName: '四处',
              name: '四处',
              id: 20,
              parentId: 1
            }, {
              children: [{ displayName: '五处一科', name: '五处一科', id: 27, parentId: 26 }, { displayName: '五处二科', name: '五处二科', id: 28, parentId: 26 },
                { displayName: '五处三科', name: '五处三科', id: 29, parentId: 26 }, { displayName: '五处四科', name: '五处四科', id: 30, parentId: 26 }, { displayName: '五处五科', name: '五处五科', id: 31, parentId: 26 }],
              displayName: '五处',
              name: '五处',
              id: 26,
              parentId: 1
            }],
          displayName: '基地',
          name: '基地',
          id: 1,
          parentId: 0
        }]
      }
      //   const res = await this.$get('/dataModel/getForceLayer')
      if (data.code === 200) {
        this.datas = data.data
        this.userwrap = data.data
      }
    },
    handleCreate () {
      this.disabled = false
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      // this.$nextTick(() => {
      //   this.$refs.dataForm.clearValidate()
      // })
      const params = {
        pageNum: 0,
        pageSize: 0,
        params: {}
      }
      this.getPowerData()

      // this.$post('/user/getRoleList', params).then(res => {
      //   this.rolewrap = []
      //   if (res.code === 200) {
      //     res.data.content.forEach((itm, inx) => {
      //       const data = { ...itm }
      //       data.id = itm.id
      //       data.key = itm.id
      //       data.label = itm.name
      //       this.rolewrap.push(data)
      //     })
      //     // console.log(this.userwrap)
      //   }
      // })
    }
  }
}
</script>

<style lang="scss" scoped>

.userManagement-wrap {
  width: 100vw;
  height: 100vh;
  .userManagement-body {
     width: 100vw;
  height: 100vh;
    padding: 20px 20px 20px 20px;
    overflow: hidden;
    scrollbar-width: 0;
  }
  .userManagement-header {
    padding: 20px 10px 0 20px;
  }
  .el-button--primary {
    background-color: #3f96a542 !important;
    border: 1px solid #3f96a5 !important;
  }
}

.userm-a {
  height: 100%;
  overflow-y: auto;
}
.user-all {
  overflow-y: auto;
  .usergroup {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    .tree-transfer {
      display: flex;
      min-height: 250px;
      margin: 40px auto;
      width: 720px;
      .tree-transfer-left,
      .tree-transfer-right {
        min-width: 300px;
        border: 1px #e5e5e5 solid;
        border-radius: 10px;
        padding: 10px;
        height: 250px;
        overflow-y: auto;
      }
      .tree-transfer-middle {
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 0 30px;
      }
    }
  }
  // .usergroup::before {
  //   content: '用户';
  //   position: relative;

  //   left: -6%;
  //   //
  // }
  .usergroup1 {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    margin-top: 20px;
  }
  // .usergroup1::before {
  //   content: '角色';
  //   position: relative;
  //   padding-left: 4px;
  //   left: -7%;
  //   //
  // }
}
</style>
<style lang='scss'>
.el-dialog {
  width: 80%;
  margin-top: 6vh!important;
}

</style>
