<template>
  <div>
    <!-- ダイアログ -->
    <v-dialog
      v-model="dialog"
      max-width="750"
      persistent
    >
      <v-card>
        <v-row no-gutters justify="center">
          <v-card-text>
            <v-data-table
              :headers="rowHeaders"
              :items="rowItems"
              item-key="name"
              disable-pagination
              hide-default-footer
              fixed-header
              height="600"
            >
              <template #[`item.value`]="{ item }">
                <v-text-field
                  v-model="item.value"
                  solo
                  flat
                  hide-details
                />
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn
              class="success"
              :disabled="!isIns"
              @click="insertData()"
            >
              登録
            </v-btn>
            &nbsp;&nbsp;
            <v-btn
              class="warning"
              :disabled="isIns"
              @click="updateData()"
            >
              更新
            </v-btn>
            &nbsp;&nbsp;
            <v-btn
              class="error"
              :disabled="isIns"
              @click="deleteData()"
            >
              削除
            </v-btn>
            &nbsp;&nbsp;
            <v-btn
              class="primary"
              @click="dialog = false"
            >
              キャンセル
            </v-btn>
            <v-spacer />
          </v-card-actions>
        </v-row>
      </v-card>
    </v-dialog>
    <v-row>
      <v-col>
        <v-card-title>
          {{ tblId }}
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
          <VueJsonToCsv
            :json-data="lists"
            :csv-title="csvFile"
          >
            <v-btn
              class="primary"
            >
              CSV
            </v-btn>
          </VueJsonToCsv>
            &nbsp;&nbsp;
          <v-btn
            class="success"
            @click="insClick()"
          >
            新規
          </v-btn>
        </v-card-actions>
      </v-col>
      <v-col>
        <v-card-actions class="justify-end">
          <v-btn
            class="info"
            @click="home()"
          >
            戻る
          </v-btn>
        </v-card-actions>
      </v-col>
    </v-row>
    <v-data-table
      :headers="headers"
      :items="lists"
      :search="search"
      item-key="line"
      @click:row="rowClick"
    />
  </div>
</template>

<script>
/* eslint-disable no-console */
export default {
  data () {
    return {
      tblId: this.$route.query.tbl,
      dialog: false,
      isIns: false,
      search: '',
      headers: [],
      lists: [],
      rowHeaders: [
        {
          text: '項目名',
          value: 'name'
        },
        {
          text: '値',
          value: 'value'
        }
      ],
      rowItems: [],
      json_meta: [
        [{
          key: 'charset',
          value: 'utf-8'
        }]
      ],
      inTblId: this.$route.query.tbl,
      csvFile: this.$route.query.tbl + '_' + new Date().toISOString().substr(0, 10)
    }
  },
  async fetch () {
    const url = 'http://localhost:3000/api/recList?tbl=' + this.inTblId
    this.headers = await fetch(url).then(res => res.json())
  },
  created () {
    if (typeof window !== 'undefined') {
      this.searchData()
    }
  },
  methods: {
    async searchData () {
      if (!this.inTblId) {
        window.alert('検索キーが未設定です！')
        return
      }
      try {
        const res = await this.$axios.$get('/api/search', {
          params: {
            tbl: this.inTblId
          }
        })
        this.lists = res
        for (const item in this.lists) {
          for (const subItem in this.lists[item]) {
            try {
              const str = this.lists[item][subItem]
              if (str !== null && str.indexOf('T15:00:00.000Z') > 0) {
                const dt = new Date(Date.parse(str))
                const month = ('0' + (dt.getMonth() + 1)).slice(-2)
                const day = ('0' + dt.getDate()).slice(-2)
                this.lists[item][subItem] = dt.getFullYear() + '-' + month + '-' + day
              }
            } catch (e) { } // 握りつぶす
          }
        }
      } catch (e) {
        console.log(e.errorCode) // eslint-disable-line no-console
        window.alert(e)
      }
    },
    rowClick (row) {
      let i = 0
      this.rowItems.splice(0)
      for (const item in row) {
        const addData = { name: this.headers[i].value, value: row[item] }
        this.rowItems.push(addData)
        i++
      }
      this.dialog = true
      this.isIns = false
    },
    insClick () {
      this.rowItems.splice(0)
      for (let i = 0; i < this.headers.length; i++) {
        const addData = { name: this.headers[i].value, value: '' }
        this.rowItems.push(addData)
      }
      this.dialog = true
      this.isIns = true
    },
    inputCheck () {
      for (let i = 0; i < this.headers.length; i++) {
        const str = this.rowItems[i].value
        const type = this.headers[i].datatype
        if (str) {
          if (type === 'varchar') {
            if (str.length > this.headers[i].dataleng) {
              window.alert(this.rowItems[i].name + 'は、[' + this.headers[i].dataleng + ']文字以内で入力してください。')
              return false
            }
          } else if (['int', 'tinyint'].includes(type)) {
            if (isNaN(str)) {
              window.alert(this.rowItems[i].name + 'が数値ではありません。')
              return false
            }
            // 数値の桁数は、COLUMN_TYPE の()の中の数字で確認できるが、数字部分取得が複雑になるので、今回そこまではしない
            // 例えば以下かと（小数点以下ありの場合が未対応で不完全）
            // if (str.length > this.headers[i].coltype.replace(/[^0-9]/g,'')) {
            //   window.alert(this.rowItems[i].name + 'が桁数オーバーしています。')
            //   return false
            // }
          } else if (type === 'date') {
            if (!str.match(/\d{4}-\d{1,2}-\d{1,2}/)) {
              window.alert(this.rowItems[i].name + 'が日付形式[yyyy-mm-dd]ではありません。')
              return false
            }
          } else {
            // 以下のアラームが出たらデータ型のチェックの追加を検討する
            window.alert(i + ' ' + type + 'は、チェック未実装')
          }
        } else if (this.headers[i].nullabl === 'NO') {
          window.alert(this.rowItems[i].name + 'は必須入力項目です。')
          return false
        }
      }
      return true
    },
    async insertData () {
      if (!this.inputCheck()) {
        return
      }
      const answer = window.confirm('登録してもいいですか？')
      if (answer) {
        try {
          await this.$axios.$post('/update?id=' + this.inTblId, this.rowItems)
          window.alert('登録処理を実行しました。')
        } catch (e) {
          console.log(e.errorCode) // eslint-disable-line no-console
          window.alert(e)
        }
      }
      this.dialog = false
      this.searchData()
    },
    async updateData () {
      if (!this.inputCheck()) {
        return
      }
      const answer = window.confirm('更新してもいいですか？')
      if (answer) {
        try {
          await this.$axios.$post('/update?id=' + this.inTblId, this.rowItems)
          window.alert('更新処理を実行しました。')
        } catch (e) {
          console.log(e.errorCode) // eslint-disable-line no-console
          window.alert(e)
        }
      }
      this.dialog = false
      this.searchData()
    },
    async deleteData () {
      const answer = window.confirm('削除してもいいですか？')
      if (answer) {
        try {
          await this.$axios.$post('/delete?id=' + this.inTblId, this.rowItems)
          window.alert('削除を実行しました。')
        } catch (e) {
          console.log(e.errorCode) // eslint-disable-line no-console
          window.alert(e)
        }
      }
      this.dialog = false
      this.searchData()
    },
    close () {
      console.log('Dialog closed') // eslint-disable-line no-console
    },
    home () {
      this.$router.push('/tableList')
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
