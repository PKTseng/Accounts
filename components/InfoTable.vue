<template>
  <div>
    <el-button
      :class="['control-delete button button-rounded  button-small  button-border filter-item', multipleSelection.length ? 'button-blue' : 'disabled']"
      type="primary"
      :disabled="multipleSelection.length === 0"
      @click="handleDelete"
    >
      {{ $t('delete') }}
    </el-button>
    <el-button
      v-permission="['Admin', 'Editor']"
      class="button button-small button-border button-rounded button-blue"
      type="primary"
      @click="handleCreate"
    >
      {{ $t('user.add') }}
    </el-button>
    <el-dialog
      :title="$t('user.addTitle')"
      :visible.sync="dialogFormVisible"
      width="35%"
      :before-close="closeForm"
    >
      <el-form
        ref="userForm"
        :model="userForm"
        :rules="rules"
        label-width="40%"
        label-position="left"
        :disabled="formDisabled"
        @submit.prevent.native
      >
        <el-form-item
          :label="$t('user.userName')"
          prop="username"
        >
          <el-input
            v-model.trim="userForm.username"
            @keyup.enter.native="dialogType === 'add' ? handleCreateUser() : updateData();"
          />
        </el-form-item>
        <el-form-item
          :label="$t('user.password')"
          prop="password"
        >
          <el-input
            v-model.trim="userForm.password"
            show-password
            @keyup.enter.native="dialogType === 'add' ? handleCreateUser() : updateData();"
          />
        </el-form-item>
        <el-form-item
          :label="$t('user.status')"
          prop="is_active"
        >
          <el-select
            v-model="userForm.is_active"
            style="width:100%"
          >
            <el-option
              label="Enabled"
              :value="true"
            />
            <el-option
              label="Disabled"
              :value="false"
            />
          </el-select>
        </el-form-item>
        <el-form-item
          :label="$t('user.role')"
          prop="role"
        >
          <el-select
            v-model="userForm.role"
            :placeholder="$t('user.selectRole')"
            style="width:100%"
          >
            <el-option
              label="Admin"
              value="Admin"
            />
            <el-option
              label="Editor"
              value="Editor"
            />
            <el-option
              label="Viewer"
              value="Viewer"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          class="button button-small button-border button-rounded"
          type="info"
          @click="dialogType === 'add' ? handleCreateUser() : updateData() "
        >
          {{ $t('save') }}
        </el-button>
        <el-button
          class="button button-small button-border button-rounded"
          @click="closeForm()"
        >
          {{ $t('close') }}
        </el-button>
      </div>
    </el-dialog>
    <el-table
      v-loading="listLoading"
      class="table"
      :data="list"
      fit
      highlight-current-row
      style="width: 100%;"
      v-bind="$store.state.settings.table.attrs"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        type="selection"
        width="50"
        :selectable="canSelectRow"
      />
      <el-table-column
        prop="username"
        :label="$t('userName')"
        align="center"
      />
      <el-table-column
        prop="role"
        :label="$t('role')"
        align="center"
      />
      <el-table-column
        prop="status"
        :label="$t('status')"
        align="center"
      >
        <template slot-scope="{ row }">
          <span v-html="row.is_active === true ? $t('user.enabled') : $t('user.disabled')" />
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        align="center"
      >
        <template slot-scope="{row,$index}">
          <el-button
            v-if="row.username !== 'admin'"
            class="button button-small button-border button-rounded"
            @click.native.prevent="handleUpdate($index, row)"
          >
            {{ $t('edit') }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="filterlist.page"
      :limit.sync="filterlist.limit"
      @pagination="getAllUser"
    />
  </div>
</template>

<script>
import { getAllUser, createUser, deleteUser, updateUser } from '@/api/user'
import Pagination from '@/components/Pagination'

export default {
  name: 'User',
  components: { Pagination },
  props: {
    filterlist: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      formDisabled: false,
      dialogType: 'add',
      listLoading: true,
      dialogFormVisible: false,
      tempIndex: undefined,
      total: 0,
      userForm: {
        username: '',
        password: '',
        role: '',
        is_active: true,
        id: undefined
      },
      list: null,
      multipleSelection: [],
      input: {}
    }
  },
  computed: {
    rules() {
      return {
        username: [
          {
            required: true,
            message: this.$t('user.formRules.userName_Empty'),
            trigger: 'blur'
          },
          { validator: this.validateSpace, trigger: 'blur' }
        ],
        password: [
          {
            min: 8,
            message: this.$t('user.formRules.password_LessChar'),
            trigger: 'blur'
          },
          {
            required: true,
            message: this.$t('user.formRules.password_Empty'),
            trigger: 'blur'
          },
          { validator: this.validatePassword, trigger: 'blur' },
          { validator: this.validateSpace, trigger: 'blur' }
        ],
        role: [
          {
            required: true,
            message: this.$t('user.formRules.role_Empty'),
            trigger: ['blur', 'change']
          }
        ]
      }
    }
  },
  watch: {
    filterlist: {
      handler() {
        this.getAllUser()
      },
      deep: true
    }
  },
  created() {
    if (Object.keys(this.$route.query).length === 0) {
      this.getAllUser()
    }
  },
  methods: {
    validatePassword(rule, value, callback) {
      if (!this.checkPassword(value)) {
        callback(new Error(this.$t('user.formRules.password_CharType')))
      } else {
        return callback()
      }
    },
    validateSpace(rule, value, callback) {
      var reg = /^[^\s]*$/
      if (!reg.test(value)) {
        callback(new Error(this.$t('user.formRules.userName_Space')))
      } else {
        return callback()
      }
    },
    checkPassword(password) {
      var checkUpperCharacter = 0
      var checkLowerCharacter = 0
      var checkNoneCharacter = 0
      if (password.match(/([A-Z])+/)) {
        checkUpperCharacter++
      }
      if (password.match(/([a-z])+/)) {
        checkLowerCharacter++
      }
      if (password.match(/([0-9])+/)) {
        checkNoneCharacter++
      }
      if (password.match(/[^a-zA-Z0-9]+/)) {
        checkNoneCharacter++
      }
      if (checkUpperCharacter > 0 && checkLowerCharacter > 0 && checkNoneCharacter > 0) {
        return true
      }
    },
    getAllUser() {
      this.listLoading = true
      getAllUser(this.filterlist)
        .then((response) => {
          this.total = response.count
          this.list = response.results.map((v) => {
            this.$set(v, 'edit', false)
            return v
          })
        })
        .finally((response) => {
          this.listLoading = false
        })
    },
    handleCreate() {
      this.userForm = {
        username: '',
        password: '',
        role: '',
        is_active: true,
        id: undefined
      }
      this.dialogFormVisible = true
      this.dialogType = 'add'
    },
    handleCreateUser() {
      this.$refs['userForm'].validate((valid) => {
        if (valid) {
          this.formDisabled = true
          this.$confirm(this.$t('createWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('no'),
            type: 'warning'
          })
            .then(() => {
              createUser(this.userForm)
                .then((response) => {
                  this.userForm.id = response.results.id
                  const tempData = Object.assign({}, this.userForm)
                  Object.assign(tempData, { edit: 'false' })
                  this.list.splice(this.list.length, 0, tempData)
                  this.list[this.list.length - 1].edit = false
                  this.dialogFormVisible = false
                  this.$notify({
                    title: this.$t('success'),
                    message: `${this.$t('createMsg')} ( ${this.userForm.username} )`,
                    type: 'success',
                    duration: 6000
                  })
                  this.$nextTick(() => {
                    this.$refs.userForm.resetFields()
                  })
                  this.$refs.userForm.resetFields()
                })
                .catch((response) => {})
            })
            .catch(() => {})
            .finally(() => {
              this.formDisabled = false
            })
        }
      })
    },
    handleUpdate(index, row) {
      this.dialogType = 'edit'
      this.dialogFormVisible = true
      this.userForm = Object.assign({}, row)
    },
    updateData() {
      this.$refs['userForm'].validate((valid) => {
        if (valid) {
          this.formDisabled = true
          this.$confirm(this.$t('editWarning'), this.$t('warning'), {
            confirmButtonText: this.$t('yes'),
            cancelButtonText: this.$t('no'),
            type: 'warning'
          })
            .then(() => {
              this.dialogFormVisible = false
              const index = this.list.findIndex((v) => v.id === this.userForm.id)
              updateUser(this.userForm).then((response) => {
                this.list.splice(index, 1, this.userForm)
              })
              this.$notify({
                title: this.$t('success'),
                message: `${this.$t('updateMsg')} ( ${this.userForm.username} )`,
                type: 'success',
                duration: 6000
              })
            })
            .finally(() => {
              this.formDisabled = false
            })
        }
      })
    },
    handleDelete() {
      this.$confirm(this.$t('deleteWarning'), this.$t('warning'), {
        confirmButtonText: this.$t('yes'),
        cancelButtonText: this.$t('no'),
        type: 'warning'
      })
        .then(() => {
          for (const ele of this.multipleSelection) {
            deleteUser(ele.id).then((response) => {
              // 判斷 ele.id 在list中的index位置
              const listIndex = this.list.findIndex((item) => item.id === ele.id)
              this.list.splice(listIndex, 1)
            })
          }
          this.$notify({
            title: this.$t('success'),
            message: this.$t('deleteMsg'),
            type: 'success',
            duration: 6000
          })
        })
        .catch(() => {})
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    closeForm() {
      this.dialogFormVisible = false
      this.$refs['userForm'].clearValidate()
    },
    canSelectRow(row, index) {
      return row.username !== 'admin'
    }
  }
}
</script>
<style lang="scss" scoped>
.table {
  margin-top: 30px;
}
</style>
