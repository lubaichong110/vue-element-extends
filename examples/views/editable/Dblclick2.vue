<template>
  <div v-loading="loading">
    <p style="color: red;font-size: 12px;">name字段（校验必填，校验最少3个字符）sex字段（校验必填）age字段（自定义校验，18-28之间）rate字段（校验最少选中3颗星）</p>
    <p style="color: red;font-size: 12px;">使用分页组件</p>

    <p>
      <el-button type="success" size="mini" @click="insertEvent()">新增一行</el-button>
      <el-button type="success" size="mini" @click="insertEvent(-1)">在最后新增一行</el-button>
      <el-button type="danger" size="mini" @click="$refs.editable.removeSelecteds()">删除选中</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.revert()">放弃更改</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.clear()">清空数据</el-button>
      <el-button type="info" size="mini" @click="$refs.editable.toggleRowExpansion($refs.editable.getRecords(1), true)">展开第二行</el-button>
      <el-button type="warning" size="mini" @click="submitEvent">校验&保存</el-button>
      <el-button type="primary" size="mini" @click="getInsertEvent">获取新增数据</el-button>
      <el-button type="primary" size="mini" @click="getUpdateEvent">获取已修改数据</el-button>
      <el-button type="primary" size="mini" @click="getRemoveEvent">获取已删除数据</el-button>
      <el-button type="primary" size="mini" @click="getSelectedEvent">获取已选中数据</el-button>
      <el-button type="primary" size="mini" @click="getAllEvent">获取所有数据</el-button>
    </p>

    <el-editable
      ref="editable"
      stripe
      border
      height="540"
      size="small"
      @select="selectEvent"
      @current-change="currentChangeEvent"
      :edit-rules="validRules"
      :edit-config="{trigger: 'dblclick', mode: 'row', showIcon: true, showStatus: true, useDefaultValidTip: true, autoScrollIntoView: true}"
      style="width: 100%">
      <el-editable-column type="selection" width="55" :selectable="selectableEvent"></el-editable-column>
      <el-editable-column type="index" width="55"></el-editable-column>
      <el-editable-column type="expand">
        <template slot-scope="props">
          <el-form label-position="left" inline>
            <el-form-item label="名称">
              <span>{{ props.row.name }}</span>
            </el-form-item>
            <el-form-item label="描述">
              <span>{{ props.row.desc }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-editable-column>
      <el-editable-column prop="name" label="名字（带校验的自定义渲染)" min-width="220" show-overflow-tooltip :edit-render="{type: 'default', autofocus: true}">
        <template slot="edit" slot-scope="scope">
          <input class="editable-custom_input" type="text" v-model="scope.row.name" @input="$refs.editable.updateStatus(scope)">
        </template>
      </el-editable-column>
      <el-editable-column prop="sex" label="性别" width="100" align="center" :edit-render="{name: 'ElSelect', options: sexList, optionProps: {value: 'value', label: 'spell'}, attrs: {clearable: true}}"></el-editable-column>
      <el-editable-column prop="age" label="年龄" width="140" align="center" headerAlign="center" :filters="ageFilterList" :filter-method="filterHandler" :edit-render="{name: 'ElInputNumber', attrs: {min: 1, max: 200}}"></el-editable-column>
      <el-editable-column prop="region" label="地区" min-width="180" :edit-render="{name: 'ElCascader', attrs: {options: regionList, separator: '->'}}"></el-editable-column>
      <el-editable-column prop="birthdate" label="日期" width="220" sortable :sort-method="birthdateSortHandler" :edit-render="{name: 'ElDatePicker', attrs: {type: 'date', format: 'yyyy年MM月dd日'}}"></el-editable-column>
      <el-editable-column prop="date1" label="选择日期" width="220" sortable :edit-render="{name: 'ElDatePicker', attrs: {type: 'datetime', format: 'yyyy-MM-dd hh:mm:ss'}}"></el-editable-column>
      <el-editable-column prop="date2" label="选择时间范围" width="260" sortable :edit-render="{name: 'ElDatePicker', attrs: {type: 'datetimerange', rangeSeparator: '至', startPlaceholder: '开始日期', endPlaceholder: '结束日期', format: 'yyyy-MM-dd'}}"></el-editable-column>
      <el-editable-column prop="color" label="选择颜色" width="100" :edit-render="{name: 'ElColorPicker', type: 'visible'}"></el-editable-column>
      <el-editable-column prop="flag2" label="是否启用2" width="200" :edit-render="{type: 'visible'}">
        <template slot="edit" slot-scope="scope">
          <el-radio-group v-model="scope.row.flag2" size="mini" @change="$refs.editable.updateStatus(scope)">
            <el-radio label="N" border>值1</el-radio>
            <el-radio label="Y" border>值2</el-radio>
          </el-radio-group>
        </template>
      </el-editable-column>
      <el-editable-column prop="status" label="状态" width="160" :edit-render="{type: 'visible'}">
        <template slot="edit" slot-scope="scope">
          <el-checkbox-group v-model="scope.row.status" size="mini" @change="$refs.editable.updateStatus(scope)">
            <el-checkbox-button label="success">成功</el-checkbox-button>
            <el-checkbox-button label="error">失败</el-checkbox-button>
          </el-checkbox-group>
        </template>
      </el-editable-column>
      <el-editable-column prop="order" label="自定义渲染" width="140" :formatter="formatterOrder" :edit-render="{type: 'default'}">
        <template slot="edit" slot-scope="scope">
          <el-autocomplete v-model="scope.row.order" :fetch-suggestions="querySearchAsync" placeholder="选中订单" @select="$refs.editable.updateStatus(scope)"></el-autocomplete>
        </template>
      </el-editable-column>
      <el-editable-column prop="remark" label="备注" min-width="180" :edit-render="{name: 'ElInput'}"></el-editable-column>
      <el-editable-column label="操作" width="220" fixed="right">
        <template slot-scope="scope">
          <template v-if="$refs.editable.hasActiveRow(scope.row)">
            <el-button size="mini" type="success" @click="saveRowEvent(scope.row)">保存</el-button>
            <el-button size="mini" type="warning" @click="cancelRowEvent(scope.row)">取消</el-button>
            <el-button size="mini" type="warning" @click="$refs.editable.revert(scope.row)">还原</el-button>
          </template>
          <template v-else>
            <el-button size="mini" type="primary" @click="$refs.editable.setActiveRow(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="removeEvent(scope.row)">删除</el-button>
            <el-button size="mini" type="warning" @click="revertEvent(scope.row)">还原</el-button>
          </template>
        </template>
      </el-editable-column>
    </el-editable>

    <el-pagination
      class="my-pagination"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pageVO.currentPage"
      :page-sizes="[5, 10, 15, 20, 50, 100, 150, 200]"
      :page-size="pageVO.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="pageVO.totalResult">
    </el-pagination>
  </div>
</template>

<script>
import XEUtils from 'xe-utils'
import { Message, MessageBox } from 'element-ui'
import listData from '@/common/json/editable/list.json'
import regionData from '@/common/json/editable/region.json'
import sexData from '@/common/json/editable/sex.json'

export default {
  data () {
    const checkAge = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('年龄不能为空'))
      }
      setTimeout(() => {
        if (!Number.isInteger(value)) {
          callback(new Error('请输入数字值'))
        } else {
          if (parseInt(value) < 18 || parseInt(value) > 28) {
            callback(new Error('年龄必须在18-28之间'))
          } else {
            callback()
          }
        }
      }, 50)
    }
    const checkRate = (rule, value, callback) => {
      if (parseInt(value) < 3) {
        callback(new Error('最小选择3颗星'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      sexList: [],
      regionList: [],
      orderDataList: [
        {
          value: '136'
        },
        {
          value: '1362'
        },
        {
          value: '13886'
        }
      ],
      ageFilterList: [
        {
          text: '26',
          value: 26
        },
        {
          text: '24',
          value: 24
        },
        {
          text: '22',
          value: 22
        }
      ],
      validRules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'change' },
          { min: 3, message: '名称长度最小 3 个字符', trigger: 'change' }
        ],
        sex: [
          { required: true, message: '请选择性别', trigger: 'blur' }
        ],
        age: [
          { validator: checkAge, trigger: 'blur' }
        ],
        rate: [
          { validator: checkRate, trigger: 'blur' }
        ]
      },
      pageVO: {
        currentPage: 1,
        pageSize: 10,
        totalResult: 0
      }
    }
  },
  created () {
    this.init()
  },
  methods: {
    init () {
      this.findList()
      this.getSexJSON().then(data => {
        this.sexList = data
      })
      this.getRegionJSON().then(data => {
        this.regionList = data
      })
    },
    findList () {
      this.loading = true
      this.getDataJSON().then(({ page, result }) => {
        this.pageVO.totalResult = page.total
        this.$refs.editable.reload(result)
        this.loading = false
      }).catch(e => {
        this.loading = false
      })
    },
    handleSizeChange (pageSize) {
      this.pageVO.pageSize = pageSize
      this.findList()
    },
    handleCurrentChange (currentPage) {
      this.pageVO.currentPage = currentPage
      this.findList()
    },
    insertEvent (index) {
      if (!this.$refs.editable.checkValid().error) {
        let row = this.$refs.editable.insertAt({ name: '默认名字1', age: 26 }, index)
        this.$nextTick(() => this.$refs.editable.setActiveRow(row))
      }
    },
    removeEvent (row) {
      MessageBox.confirm('确定删除该数据?', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$refs.editable.remove(row)
      }).catch(e => e)
    },
    revertEvent (row) {
      if (this.$refs.editable.hasRowChange(row)) {
        MessageBox.confirm('确定还原该行数据?', '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$refs.editable.revert(row)
          Message({ message: '数据还原成功！', type: 'success' })
        }).catch(e => e)
      } else {
        Message({ message: '数据未改动！', type: 'info' })
      }
    },
    saveRowEvent (row) {
      this.$refs.editable.validateRow(row).then(valid => {
        this.postJSON('url', { row }).then(data => {
          Message({ message: '保存成功', type: 'success' })
        })
      }).catch(e => {
        console.log('error row submit!!')
      })
    },
    cancelRowEvent (row) {
      this.$refs.editable.validateRow(row, (valid, validErrs) => {
        if (valid) {
          this.$refs.editable.clearActive()
        } else {
          let message = <p>
            <p>请正确填写以下信息！</p>
            {
              Object.keys(validErrs).map(name => {
                let errors = validErrs[name]
                let msg = `${name}：${errors.map(e => e.message).join(';')}`
                return <p>{msg}</p>
              })
            }
          </p>
          Message({ message, dangerouslyUseHTMLString: true, type: 'error' })
        }
      })
    },
    selectEvent (selection, row) {
      console.log(selection)
    },
    currentChangeEvent (currentRow, oldCurrentRow) {
      console.log(oldCurrentRow)
    },
    formatterOrder (row, column, cellValue, index) {
      return `订单号：${cellValue}`
    },
    submitEvent () {
      this.$refs.editable.validate(valid => {
        if (valid) {
          let { insertRecords, removeRecords, updateRecords } = this.$refs.editable.getAllRecords()
          this.postJSON('url', { insertRecords, removeRecords, updateRecords }).then(data => {
            this.findList()
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getInsertEvent () {
      let rest = this.$refs.editable.getInsertRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取新增数据(${rest.length}条)` }).catch(e => e)
    },
    getUpdateEvent () {
      let rest = this.$refs.editable.getUpdateRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取已修改数据(${rest.length}条)` }).catch(e => e)
    },
    getRemoveEvent () {
      let rest = this.$refs.editable.getRemoveRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取已删除数据(${rest.length}条)` }).catch(e => e)
    },
    getSelectedEvent () {
      let rest = this.$refs.editable.getSelecteds()
      MessageBox({ message: JSON.stringify(rest), title: `获取已选中数据(${rest.length}条)` }).catch(e => e)
    },
    getAllEvent () {
      let rest = this.$refs.editable.getRecords()
      MessageBox({ message: JSON.stringify(rest), title: `获取所有数据(${rest.length}条)` }).catch(e => e)
    },
    selectableEvent (row, index) {
      return index >= 1
    },
    birthdateSortHandler (a, b) {
      return -1
    },
    filterHandler (value, row, column) {
      return row[column.property] === value
    },
    querySearchAsync (queryString, cb) {
      var orderDataList = this.orderDataList
      var results = queryString ? orderDataList.filter(this.createStateFilter(queryString)) : orderDataList
      clearTimeout(this.timeout)
      this.timeout = setTimeout(() => {
        cb(results)
      }, 3000 * Math.random())
    },
    createStateFilter (queryString) {
      return (state) => {
        return (state.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0)
      }
    },
    postJSON (data) {
      // 提交请求
      return new Promise(resolve => {
        setTimeout(() => {
          resolve('保存成功')
        }, 300)
      })
    },
    getSexJSON () {
      // 模拟数据
      return new Promise(resolve => {
        setTimeout(() => resolve(XEUtils.clone(sexData, true)), 100)
      })
    },
    getDataJSON () {
      // 模拟分页数据
      return new Promise(resolve => {
        let list = []
        Array.from(new Array(50)).map(item => {
          list = list.concat(listData)
        })
        list = XEUtils.shuffle(XEUtils.clone(list, true))
        list.length = this.pageVO.pageSize
        setTimeout(() => resolve({
          page: {
            total: 1000
          },
          result: list
        }), 350)
      })
    },
    getRegionJSON () {
      // 模拟数据
      return new Promise(resolve => {
        setTimeout(() => resolve(XEUtils.clone(regionData, true)), 200)
      })
    }
  }
}
</script>

<style scoped>
.my-pagination {
  margin: 15px 20px 0 0;
  text-align: right;
}
.editable-custom_input {
  width: 95%;
}
</style>
