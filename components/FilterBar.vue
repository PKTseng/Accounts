<template>
  <el-form
    ref="form"
    style="margin-bottom: 10px"
    @submit.prevent.native
  >
    <el-select
      v-model="filter.role"
      class="filter-item"
      multiple
      collapse-tags
      filterable
      clearable
      :placeholder="$t('user.filter.rolePlaceholder')"
      width="100px"
      @change="handleSelectAll('role')"
    >
      <el-option
        :key="$t('all')"
        :label="$t('all')"
        :value="$t('all')"
      />
      <el-option
        v-for="item in selection.role"
        :key="item"
        :label="item"
        :value="item"
      />
    </el-select>
    <el-input
      v-model.trim="filter.username"
      :placeholder="$t('user.filter.userPlaceholder')"
      style="width: 200px;"
      class="filter-item"
      clearable
      @keyup.enter.native="handleFilter"
    />
    <el-button
      class="button button-small button-border button-rounded"
      type="info"
      @click="handleFilter"
    >
      {{ $t('search') }}
    </el-button>
  </el-form>
</template>
<script>
import waves from '@/directive/waves' // waves directive
import { getAllUser } from '@/api/user'
import selectAll from '@/views/modules/filterBar/selectAll'
import isSelectAll from '@/views/modules/filterBar/isSelectAll'
import handleQueryParamSelect from '@/views/modules/filterBar/handleQueryParamSelect'

export default {
  name: 'FilterBar',
  directives: { waves },
  props: {
    filterlist: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      filter: {
        role: [],
        username: ''
      },
      selectAllOption: {
        role: false
      },
      selection: {
        role: ['Admin', 'Editor', 'Viewer']
      }
    }
  },
  async created() {
    await getAllUser()
    await this.handleQueryParam()
  },
  methods: {
    handleQueryParamSelect,
    selectAll,
    isSelectAll,
    handleFilter() {
      this.filterlist.page = 1
      this.filterlist.role = this.isSelectAll(this.filter.role)
      this.filterlist.username = this.filter.username.toString()
      const urlPath = `/accounts?role=${this.filterlist.role}&username=${this.filterlist.username}`
      history.pushState({}, '', urlPath)
    },
    async handleQueryParam() {
      // 判斷route.query有沒有值，有值的話就填寫到filter裡
      if (Object.keys(this.$route.query).length !== 0) {
        const queryParam = {
          role: this.$route.query.role
        }
        const username = this.$route.query.username
        this.filter.role = queryParam.role || ''
        this.filter.username = username || ''
        for (var field in queryParam) {
          let payload = {
            data: queryParam[field],
            selectAllOption: this.selectAllOption[field],
            filter: this.filter[field],
            selection: this.selection[field]
          }
          payload = this.handleQueryParamSelect(payload)
          this.selectAllOption[field] = payload.selectAllOption
          this.filter[field] = payload.filter
          this.selection[field] = payload.selection
        }
        this.filterlist.role = this.isSelectAll(queryParam.role)
        this.filterlist.username = username
      }
    },
    handleSelectAll(field) {
      let payload = {
        selectAllOption: this.selectAllOption[field],
        filter: this.filter[field],
        selection: this.selection[field]
      }
      payload = selectAll(payload)
      this.selectAllOption[field] = payload.selectAllOption
      this.filter[field] = payload.filter
      this.selection[field] = payload.selection
    }
  }
}
</script>
<style lang="scss" scoped>
.filter-item {
  margin: 0 10px 10px 0;
}
</style>
