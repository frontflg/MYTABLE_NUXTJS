<template>
  <div>
    <v-row>
      <v-col>
        <v-card-title>
          TABLELIST
          <v-spacer />
          <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
          />
        </v-card-title>
      </v-col>
      <v-col>
        <v-card-actions>
          <v-btn
            class="primary"
            @click="downloadData()"
          >
            CSV
          </v-btn>
        </v-card-actions>
      </v-col>
    </v-row>
    <v-data-table
      :headers="headers"
      :items="lists"
      :search="search"
      item-key="line"
    >
      <template #[`item.TABLE_NAME`]="{ item }">
        <a :href="`http://localhost:3000/recList?tbl=${item.TABLE_NAME}`">
          <div style="font-size: 130%">{{ item.TABLE_NAME }}</div>
        </a>
      </template>
      <template #[`item.CREATE_TIME`]="{ item }">
        {{ item.CREATE_TIME.slice(0,10) }}
      </template>
    </v-data-table>
  </div>
</template>

<script>
/* eslint-disable no-console */
export default {
  name: 'TableList',
  async asyncData ({ $axios }) {
    try {
      const lists = await $axios.$get('/api')
      return { lists }
    } catch (e) {
      console.log(e.errorCode) // eslint-disable-line no-console
      window.alert(e)
    }
  },
  data () {
    return {
      search: '',
      headers: [
        {
          text: 'テーブルID',
          value: 'TABLE_NAME'
        },
        {
          text: 'テーブル名',
          value: 'TABLE_COMMENT'
        },
        {
          text: '件数',
          value: 'TABLE_ROWS'
        },
        {
          text: '作成日',
          value: 'CREATE_TIME'
        }
      ],
      lists: []
    }
  },
  methods: {
    downloadData () {
      let csv = 'テーブルID,テーブル名,件数,作成日\n'
      this.lists.forEach(function (el) {
        csv += el.TABLE_NAME + ',' + el.TABLE_COMMENT + ',' + el.TABLE_ROWS + ',' + el.CREATE_TIME + '\n'
      })
      const anchor = document.createElement('a')
      anchor.href = 'data:text/csv;charset=utf-8,' + encodeURIComponent(csv)
      anchor.target = '_blank'
      anchor.download = 'TABLE_LIST_' + new Date().toISOString().substr(0, 10) + '.csv'
      anchor.click()
    }
  }
}
</script>

<style lang="scss">
.v-data-table th {
  background: #8C9EFF;
}
.v-data-table td {
  background: #E8F5E9;
  border: 1px #BDBDBD solid;
}
.v-data-table tr:nth-child(odd) td {
  background: #F5F5F5;
}
.v-data-table tr:hover td {
  background-color: #FFFF8D;
}
</style>
