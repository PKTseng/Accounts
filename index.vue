<template>
  <div>
    <h1 class="page-title">{{ pageTitle }}</h1>
    <div class="app-container">
      <filter-bar
        v-if="initInfoTable"
        :filterlist="filterlist"
      />
      <info-table
        :filterlist="filterlist"
        @hook:mounted="initInfoTable = true"
      />
    </div>
  </div>
</template>

<script>
import FilterBar from './components/FilterBar'
import InfoTable from './components/InfoTable'

export default {
  name: 'Accounts',
  components: {
    FilterBar,
    InfoTable
  },
  data() {
    return {
      pageTitle: this.$t('user.pageTitle'),
      filterlist: {
        page: 1,
        limit: 20,
        role: null,
        username: null
      },
      initInfoTable: false
    }
  }
}
</script>
