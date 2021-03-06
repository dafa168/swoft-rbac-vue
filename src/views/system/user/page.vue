<template>
  <d2-container>
    <template slot="header">
      <el-form
        :inline="true"
        :model="searchForm"
        ref="searchForm"
        size="mini"
        style="margin-bottom: -18px;"
      >
        <el-form-item
          label="名称"
          prop="name"
        >
          <el-input
            v-model="searchForm.name"
            placeholder="名称"
            style="width: 100px;"
          />
        </el-form-item>
        <el-form-item
          label="邮箱"
          prop="email"
        >
          <el-input
            v-model="searchForm.email"
            placeholder="邮箱"
            style="width: 120px;"
          />
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            @click="handleSearchFormSubmit"
          >
            <d2-icon name="search" /> 查询
          </el-button>
        </el-form-item>
        <el-form-item>
          <el-button @click="handleSearchFormReset">
            <d2-icon name="refresh" /> 重置
          </el-button>
        </el-form-item>
      </el-form>
    </template>
    <el-button
      type="primary"
      size="mini"
      icon="el-icon-circle-plus"
      @click="add"
    >
      新增
    </el-button>
    <el-button
      v-if="multipleSelection.length>0"
      type="danger"
      size="mini"
      icon="el-icon-delete"
      @click="batchDel"
    >
      删除
    </el-button>
    <el-table
      :data="tableData"
      v-loading="loading"
      size="small"
      stripe
      highlight-current-row
      style="width: 100%;"
      @selection-change="handleSelectionChange"
      @sort-change="handleSortChange"
    >

      <el-table-column
        type="selection"
        width="55"
      >
      </el-table-column>
      <el-table-column
        label="账号"
        prop="username"
        sortable
      >
        <template slot-scope="scope">
          {{scope.row.username}}
        </template>
      </el-table-column>
      <el-table-column
        label="姓名"
        prop="realName"
        sortable
      >
        <template slot-scope="scope">
          {{scope.row.realName}}
        </template>
      </el-table-column>
      <el-table-column
        label="说明"
        prop="description"
        :show-overflow-tooltip="true"
      >
        <template slot-scope="scope">
          {{scope.row.description}}
        </template>
      </el-table-column>
      <el-table-column
        label="邮箱"
        prop="email"
        sortable
        :show-overflow-tooltip="true"
      >
        <template slot-scope="scope">
          {{scope.row.email}}
        </template>
      </el-table-column>
      <el-table-column
        label="mobile"
        :show-overflow-tooltip="true"
      >
        <template slot-scope="scope">
          {{scope.row.mobile}}
        </template>
      </el-table-column>
      <el-table-column
        label="性别"
        :show-overflow-tooltip="true"
      >
        <template slot-scope="scope">
          {{scope.row.sex == 1 ? '女': (scope.row.sex == 2 ? '保密' : '男')}}
        </template>
      </el-table-column>
      <el-table-column
        label="状态"
        :show-overflow-tooltip="true"
      >
        <template slot-scope="scope">
          <div v-if="scope.row.status == 0"><el-tag type="warning">锁定</el-tag></div>
          <div v-else><el-tag>有效</el-tag></div>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        label="操作"
        align="center"
      >
        <template slot-scope="scope">
          <el-button
            type="primary"
            title="编辑"
            size="mini"
            icon="el-icon-edit"
            circle
            @click="openEditForm(scope.row)"
          ></el-button>
          <el-button
            type="danger"
            title="删除"
            size="mini"
            icon="el-icon-delete"
            circle
            @click="del(scope.row.id)"
          ></el-button>
          <el-button
            type="warning"
            title="角色列表"
            size="mini"
            icon="el-icon-share"
            circle
            @click="openUserRoleDialog(scope.row)"
          ></el-button>
        </template>
      </el-table-column>

    </el-table>
    <template slot="footer">
      <el-pagination
        :current-page="page.current"
        :page-size="page.size"
        :total="page.total"
        :page-sizes="[1,100, 200, 300, 400]"
        layout="total, sizes, prev, pager, next, jumper"
        style="margin: -10px;"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      >
      </el-pagination>
    </template>
    <user-edit
      :user="user"
      v-model="editFormVisible"
      @submit="getTableData"
    />
    <user-role
      :user="user"
      v-model="userRoleDialogVisible"
    />
  </d2-container>
</template>
<script>
import * as userService from '@/api/sys/user'
import UserRole from './components/user-role'
import UserEdit from './components/user-edit'
export default {
  name: 'UserPage',
  components: { UserEdit, UserRole },
  data () {
    return {
      searchForm: {
        name: '',
        email: ''
      },
      loading: false,
      tableData: [],
      multipleSelection: [],
      page: {
        current: 1,
        size: 100,
        total: 0
      },
      sort: {
        prop: '',
        order: ''
      },
      user: {},
      editFormVisible: false,
      userRoleDialogVisible: false
    }
  },
  mounted () {
    this.getTableData()
  },
  methods: {
    getTableData () {
      let query = {
        page: this.page.current,
        pageSize: this.page.size,
        sortBy: this.sort.prop,
        descending: this.sort.order === 'descending',
        filter: this.searchForm
      }
      userService.getUserList(query).then(res => {
        this.tableData = res.data
        this.page.total = res.total
      })
    },
    handleSearchFormSubmit () {
      this.getTableData()
    },
    handleSearchFormReset () {
      this.$refs.searchForm.resetFields()
    },
    handleSelectionChange (val) {
      this.multipleSelection = val
    },
    handleSortChange (val) {
      this.sort.prop = val.prop
      this.sort.order = val.order
      this.getTableData()
    },
    handleSizeChange (val) {
      this.page.size = val
      this.getTableData()
    },
    handleCurrentChange (val) {
      this.page.current = val
      this.getTableData()
    },
    openEditForm (user) {
      this.user = user
      this.editFormVisible = true
    },
    add () {
      this.user = {}
      this.editFormVisible = true
    },
    openUserRoleDialog (user) {
      this.user = user
      this.userRoleDialogVisible = true
    },
    batchDel () {
      this.$confirm('确认删除？', '确认信息', {
        distinguishCancelAndClose: true,
        confirmButtonText: '删除',
        cancelButtonText: '取消'
      }).then(() => {
        userService
          .delUsers(this.multipleSelection.map(s => s.id))
          .then(() => {
            this.getTableData()
          })
      })
    },
    del (id) {
      this.$confirm('确认删除？', '确认信息', {
        distinguishCancelAndClose: true,
        confirmButtonText: '删除',
        cancelButtonText: '取消'
      }).then(() => {
        userService.delUsers([id]).then(() => {
          this.getTableData()
        })
      })
    }
  }
}
</script>
