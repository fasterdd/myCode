<template>
  <div>
    <Table :data="table" ref="table">
      <!-- <template v-slot:tag_slot="scope">
            <el-tag :type="scope.row.tag=='家'?'success':'danger'">
                {{scope.row.tag}}
            </el-tag>
        </template>-->

      <template v-slot:operate_slot="scope">
        <el-button type="warning" @click="edit(scope.row)">编辑</el-button>
        <el-popconfirm title="你确定要删除吗？" @confirm="del(scope.row)">
          <el-button type="danger" slot="reference" style="margin-left: 10px"
            >删除</el-button
          >
        </el-popconfirm>
      </template>

      <!--<template v-slot:href_slot="scope">
            <p v-html="scope.row.href"></p>
        </template>-->

      <template v-slot:goodstype_slot="scope">
        {{ goodsttypeFormat(scope.row) }}
      </template>

      <template v-slot:storage_slot="scope">
        {{ storageFormat(scope.row) }}
      </template>

      <!-- 搜索下拉插槽-->
      <template v-slot:storage_search_slot>
        <SelectComp
          :select="select_storage"
          v-model="select_storage_val"
        ></SelectComp>
      </template>

      <template v-slot:goodstype_search_slot>
        <SelectComp
          :select="select_goodstype"
          v-model="select_goodstype_val"
        ></SelectComp>
      </template>

      <template v-slot:button_slot="scope">
        <el-button type="primary" @click="add()">新增</el-button>
        <el-popconfirm title="你确定要删除吗？" @confirm="delBatch(scope.data)">
          <el-button
            type="danger"
            slot="reference"
            style="margin-left: 10px"
            :disabled="scope.data.length <= 0"
            >批量删除</el-button
          >
        </el-popconfirm>
        <el-button
          type="primary"
          @click="outGoods"
          style="margin-left: 10px"
          :disabled="scope.data.length != 1"
          >出货</el-button
        >
        <el-button
          type="primary"
          @click="inGoods"
          :disabled="scope.data.length != 1"
          >补货</el-button
        >
      </template>

      <template v-slot:form>
        <el-dialog title="物品维护" :visible.sync="table.formShowFlag" center>
          <Form :data="formData" ref="form">
            <!-- 表单下拉插槽-->
            <template v-slot:storage_form_slot>
              <SelectComp
                :select="select_storage"
                v-model="storage_form_val"
              ></SelectComp>
            </template>
            <template v-slot:goodstype_form_slot>
              <SelectComp
                :select="select_goodstype"
                v-model="goodstype_form_val"
              ></SelectComp>
            </template>
          </Form>
        </el-dialog>
      </template>

      <template v-slot:formRecord>
        <el-dialog
          :title="formRecordData.title"
          :visible.sync="formRecordShowFlag"
          center
        >
          <Form :data="formRecordData" ref="formRecord">
            <!-- 表单下拉插槽-->
            <template v-slot:user_form_slot>
              <el-input
                v-model="select_userName"
                readonly
                placeholder="请选择用户"
                @click.native="showSelect"
                style="width: 240px"
              ></el-input>
            </template>
          </Form>
        </el-dialog>
      </template>
    </Table>

    <el-dialog
      title="选择用户"
      :visible.sync="userSelectTableShow"
      @close="close"
      width="860px"
    >
      <SelectTable
        :select="selectTable"
        v-if="selectTableShow"
        ref="refSelectTable"
      >
      </SelectTable>
    </el-dialog>
  </div>
</template>

<script>
import Table from './TableComp'
import SelectComp from '../select/SelectComp'
import Form from '../formComp/Form'
import SelectTable from '../../selectTest/SelectTable'

export default {
  name: 'TestTable',
  components: { SelectComp, Table, Form, SelectTable },
  data() {
    let checkCount = (rule, value, callback) => {
      if (value > 9999) {
        callback(new Error('数量输入过大'))
      } else {
        callback()
      }
    }

    return {
      storageData: [],
      goodstypeData: [],
      select_storage_val: '',
      select_storage: {
        name: '仓库',
        data: [],
      },
      select_goodstype_val: '',
      select_goodstype: {
        name: '分类',
        data: [],
      },
      table: {
        http: {
          url: 'http://localhost:8090/goods/listByPage',
          result: {
            list: 'data.list',
            total: 'data.total',
          },
        },
        pageSize: 10,

        formShowFlag: false,
        forms: [
          {
            name: '商品维护',
            slot_name: 'form',
          },
          {
            name: '出货补货',
            slot_name: 'formRecord',
          },
        ],
        search: [
          {
            label: '物品名',
            prop: 'name',
            width: '180',
            placeholder: '请输入物品名',
            default: '',
          },
          {
            type: 'slot',
            slot_name: 'storage_search_slot',
            label: '仓库',
            prop: 'storage',
            width: '180',
            placeholder: '请选择仓库',
            default: '',
          },

          {
            type: 'slot',
            slot_name: 'goodstype_search_slot',
            label: '仓库',
            prop: 'goodstype',
            width: '180',
            placeholder: '请选择分类',
            default: '',
          },

          {
            label: '角色',
            prop: 'role',
            default: 1,
            static: true,
          },
        ],
        columns: [
          {
            label: 'ID',
            prop: 'id',
            width: '80',
            //hide:true
          },
          {
            label: '物品名',
            prop: 'name',
            width: '180',
          },
          {
            type: 'slot',
            slot_name: 'storage_slot',
            label: '仓库',
            prop: 'storage',
            width: '180',
          },
          {
            type: 'slot',
            slot_name: 'goodstype_slot',
            label: '类型',
            prop: 'goodstype',
            width: '180',
          },
          {
            label: '数量',
            prop: 'count',
            width: '180',
          },
          {
            label: '备注',
            prop: 'remark',
          },
          /* {
                            type:'slot',
                            slot_name:'tag_slot',
                            label:'Tag',
                            prop:'tag',
                            width:'180'
                        },*/
          {
            type: 'slot',
            slot_name: 'operate_slot',
            label: '操作',
            prop: 'operate',
            width: '180',
          },
        ],
        abc: '',
      },

      storage_form_val: '',
      goodstype_form_val: '',
      formData: {
        width: '120px',
        items: [
          {
            type: 'Input',
            label: 'ID',
            prop: 'id',
            hide: true,
          },
          {
            type: 'Input',
            label: '物品名',
            prop: 'name',
            width: '240px',
            placeholder: '请填写物品名',
            default: '',
            readonly: false,
          },
          {
            type: 'slot',
            slot_name: 'storage_form_slot',
            label: '仓库',
            prop: 'storage',
            width: '240px',
            placeholder: '请选择仓库',
          },
          {
            type: 'slot',
            slot_name: 'goodstype_form_slot',
            label: '分类',
            prop: 'storage',
            width: '240px',
            placeholder: '请选择分类',
          },
          {
            type: 'Input',
            label: '数量',
            prop: 'count',
            width: '240px',
            placeholder: '请填写数量',
            default: '',
            readonly: false,
          },
          {
            type: 'Textarea',
            label: '备注',
            prop: 'remark',
            span: 18,
            placeholder: '请填写备注',
          },
        ],
        buttons: [
          {
            label: '确定',
            type: 'primary',
            action: 'submit',
            call: (data) => {
              console.log(data)
              //提交表单
              this.submit(data)
            },
          },
          {
            label: '重置',
            type: 'primary',
            action: 'reset',
            call: () => {
              console.log('reset')

              this.storage_form_val = ''
              this.goodstype_form_val = ''
            },
          },
        ],
        rules: {
          name: [{ required: true, message: '请输入物品名', trigger: 'blur' }],
          storage: [{ required: true, message: '请选中仓库', trigger: 'blur' }],
          goodstype: [
            { required: true, message: '请选择分类', trigger: 'blur' },
          ],
          count: [
            { required: true, message: '请输入数量', trigger: 'blur' },
            {
              pattern: /^([1-9][0-9]*){1,4}$/,
              message: '数量必须为正整数字',
              trigger: 'blur',
            },
            { validator: checkCount, trigger: 'blur' },
          ],
        },
      },

      select_userName: '',
      formRecordShowFlag: false,
      formRecordData: {
        title: '',
        width: '120px',
        items: [
          {
            type: 'Input',
            label: 'ID',
            prop: 'id',
            hide: true,
          },
          {
            type: 'Input',
            label: '物品ID',
            prop: 'goods',
            hide: true,
          },
          {
            type: 'Input',
            label: '物品名',
            prop: 'name',
            width: '240px',
            placeholder: '请填写物品名',
            default: '',
            readonly: true,
          },
          {
            type: 'Input',
            prop: 'userid',
            default: '',
            hide: true,
          },
          {
            type: 'slot',
            slot_name: 'user_form_slot',
            label: '出货/补货人',
            prop: 'userName',
            width: '240px',
            placeholder: '请选择出货/补货人',
          },
          {
            type: 'Input',
            label: '数量',
            prop: 'count',
            width: '240px',
            placeholder: '请填写数量',
            default: '',
            readonly: false,
          },
          {
            type: 'Textarea',
            label: '备注',
            prop: 'remark',
            span: 18,
            placeholder: '请填写备注',
          },
          {
            type: 'Input',
            prop: 'adminId',
            default: '',
            hide: true,
          },
          {
            type: 'Input',
            prop: 'adminName',
            default: '',
            hide: true,
          },
          {
            type: 'Input',
            prop: 'action',
            default: '',
            hide: true,
          },
        ],
        buttons: [
          {
            label: '确定',
            type: 'primary',
            action: 'submit',
            call: (data) => {
              console.log(data)
              //提交表单
              this.submitRecord(data)
            },
          },
          {
            label: '重置',
            type: 'primary',
            action: 'reset',
            call: () => {
              console.log('reset')
              this.select_userName = ''
            },
          },
        ],
        rules: {
          userName: [{ required: true, message: '请选择用户' }],
          count: [
            { required: true, message: '请输入数量', trigger: 'blur' },
            {
              pattern: /^([1-9][0-9]*){1,4}$/,
              message: '数量必须为正整数字',
              trigger: 'blur',
            },
            { validator: checkCount, trigger: 'blur' },
          ],
        },
      },
      userSelectTableShow: false,
      selectTableShow: false,
      selectTable: {
        url: '/user/listByPage',

        res: {
          data: 'data.list',
          total: 'data.total',
        },
        //默认查询参数
        defaultParam: {
          roleId: 2, //用户
        },
        //resultSize:2,
        //表示单选
        //single:true,
        //确定的回调
        submit: (data) => {
          console.log(data)
          this.userSelectTableShow = false

          let id, name

          if (!Array.isArray(data)) {
            name = data.name
            id = data.id
          } else {
            name = data[0].name
            id = data[0].id
          }

          this.$refs.formRecord.setForm({ userid: id, userName: name })

          this.select_userName = name
        },
        //取消的回调
        cancel: () => {
          this.userSelectTableShow = false
        },
        columns: [
          {
            prop: 'id',
            label: 'ID',
            key: true,
            show: true,
          },
          {
            prop: 'name',
            label: '名字',
            width: 200,
            search: true,
          },
          {
            prop: 'no',
            label: '账号',
            width: 200,
            //search:true
          },
          {
            prop: 'age',
            label: '年龄',
            width: 80,
          },
          {
            prop: 'phone',
            label: '电话',
            // return:true
          },
        ],
      },
    }
  },
  methods: {
    add() {
      this.table.formShowFlag = true
      this.$nextTick(() => {
        this.$refs.form.resetForm()
        this.storage_form_val = ''
        this.goodstype_form_val = ''
      })
    },
    edit(row) {
      console.log(row)
      this.table.formShowFlag = true
      this.$nextTick(() => {
        this.$refs.form.setForm(row)

        this.storage_form_val = row.storage + ''
        this.goodstype_form_val = row.goodstype + ''
      })
    },
    del(row) {
      console.log(row)

      this.http
        .post('http://localhost:8090/goods/del', { id: row.id })
        .then((res) => {
          console.log(res)
          if (res.code == 200) {
            //成功
            this.$message({
              showClose: true,
              message: '操作成功！',
              type: 'success',
            })

            this.$refs.table.load()
          } else {
            this.$message({
              showClose: true,
              message: '操作失败！',
              type: 'error',
            })
          }
        })
    },
    delBatch(data) {
      console.log(data)
      //这里写删除的代码
      let ids = ''
      data.forEach(function (item) {
        ids += item.id + ',' // 添加所有需要删除数据的id到一个数组，post提交过去
      })

      console.log('ids===' + ids)
      this.http
        .post(
          '/goods/delBatch',
          JSON.stringify(ids.substr(0, ids.length - 1).split(','))
        )
        .then((res) => {
          console.log(res)
          if (res.code == 200) {
            //成功
            this.$message({
              showClose: true,
              message: '操作成功！',
              type: 'success',
            })
            //重新加载子组件的数据
            this.$refs.table.load()
          } else {
            this.$message({
              showClose: true,
              message: '操作失败！',
              type: 'error',
            })
          }
        })
    },
    loadStorage() {
      this.http.post('/storage/listAll', {}).then((res) => {
        this.storageData = res.data
        this.select_storage.data = res.data
      })
    },
    loadGoodstype() {
      this.http.post('/goodstype/listAll', {}).then((res) => {
        this.goodstypeData = res.data
        this.select_goodstype.data = res.data
      })
    },
    storageFormat(row) {
      let temp = this.storageData.find((item) => {
        return item.value == row.storage
      })
      return temp && temp.label
    },
    goodsttypeFormat(row) {
      let temp = this.goodstypeData.find((item) => {
        return item.value == row.goodstype
      })
      return temp && temp.label
    },
    submit(data) {
      this.http.post('/goods/save', data).then((res) => {
        console.log(res)
        if (res.code == 200) {
          //成功
          this.$message({
            showClose: true,
            message: '操作成功！',
            type: 'success',
          })
          this.table.formShowFlag = false
          //this.$refs.form.resetFields();

          this.$refs.table.load()
        } else {
          this.$message({
            showClose: true,
            message: '操作失败！',
            type: 'error',
          })
        }
      })
    },
    submitRecord(data) {
      this.http.post('/record/save', data).then((res) => {
        console.log(res)
        if (res.code == 200) {
          //成功
          this.$message({
            showClose: true,
            message: '操作成功！',
            type: 'success',
          })
          this.formRecordShowFlag = false
          this.$refs.formRecord.resetForm()
          this.$refs.table.load()
        } else {
          this.$message({
            showClose: true,
            message: '操作失败！',
            type: 'error',
          })
        }
      })
    },
    outGoods() {
      this.formRecordShowFlag = true
      this.formRecordData.title = '出货'

      let row = this.$refs.table.multipleSelection[0]
      let o = {
        adminName: '超级管理',
        adminId: '1',
        name: row.name,
        goods: row.id,
        action: 'out',
      }
      this.$nextTick(() => {
        this.$refs.formRecord.setForm(o)
      })
    },
    inGoods() {
      this.formRecordShowFlag = true
      this.formRecordData.title = '补货'

      let row = this.$refs.table.multipleSelection[0]
      let o = {
        adminName: '超级管理',
        adminId: '1',
        name: row.name,
        goods: row.id,
        action: 'in',
      }
      this.$nextTick(() => {
        this.$refs.formRecord.setForm(o)
      })
    },
    showSelect() {
      this.userSelectTableShow = true

      this.selectTableShow = false

      this.$nextTick(() => {
        this.selectTableShow = true
      })
    },
    close() {
      this.userSelectTableShow = false
    },
  },
  created() {
    this.loadStorage()
    this.loadGoodstype()
  },
  watch: {
    select_storage_val: {
      handler(newVal) {
        console.log(newVal)

        let o = { storage: newVal }
        this.$refs.table.setSearchForm(o)
      },
    },
    select_goodstype_val: {
      handler(newVal) {
        console.log(newVal)

        let o = { goodstype: newVal }
        this.$refs.table.setSearchForm(o)
      },
    },
    storage_form_val: {
      handler(newVal) {
        console.log(newVal)

        let o = { storage: newVal }
        this.$refs.form.setForm(o)
      },
    },
    goodstype_form_val: {
      handler(newVal) {
        console.log(newVal)

        let o = { goodstype: newVal }
        this.$refs.form.setForm(o)
      },
    },
  },
}
</script>

<style scoped></style>
