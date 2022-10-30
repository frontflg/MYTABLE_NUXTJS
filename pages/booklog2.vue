<template>
  <div>
    <!-- ダイアログ：ISBN13入力 -->
    <v-dialog
      v-model="dialog"
      max-width="400"
      persistent
    >
      <v-card>
        <v-row no-gutters justify="center">
          <v-card-title>
            ISBN13入力
          </v-card-title>
          <v-card-text>
            <v-text-field
              v-model="inIsbn13"
              label="ISBN13："
              maxlength="13"
              class="mx-5"
              clearable
            />
            を登録しますか？
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn
              class="primary"
              @click="insertData()"
            >
              ＯＫ
            </v-btn>
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
      <v-col cols="10">
        <v-card-title>
          BOOKLOG
          <v-spacer />
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
          <v-btn
            class="success"
            @click="dialogOpen()"
          >
            登録
          </v-btn>
        </v-card-actions>
      </v-col>
    </v-row>
    <v-row dense class="toc-view" width="100%" max-height="350">
      <v-col
        v-for="(item, i) in lists"
        :key="i"
        cols="12"
        lg="2"
      >
        <v-card
          color="grey lighten-4"
          max-width="100%"
          height="90%"
        >
          <v-card-actions class="mb-5">
            <a :href="`http://localhost:3000/book?id=${item.ISBN13}`">
              <v-img :src="`https://images-na.ssl-images-amazon.com/images/I/${item.CoverImg}`" max-width="175" />
            </a>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
/* eslint-disable no-console */
export default {
  name: 'BooklogList',
  async asyncData ({ $axios }) {
    try {
      // const lists = await $axios.$get('http://localhost:3000/api/booklog')
      const lists = await $axios.$get('/api/booklog')
      return { lists }
    } catch (e) {
      console.log(e.errorCode) // eslint-disable-line no-console
      window.alert(e)
    }
  },
  data () {
    return {
      search: '',
      dialog: false,
      inIsbn13: '',
      selRow: 0,
      lists: [],
      imgUrl: ''
    }
  },
  methods: {
    downloadData () {
      let csv = '\uFEFF' + 'ISBN13,書籍名,著者,出版社,価格,分類,発行日\n'
      this.lists.forEach(function (el) {
        csv += el.ISBN13 + ',' + el.BookName + ',' + el.Author + ',' +
        el.Publisher + ',' + el.Purchase + ',' +
        el.Genre + ',' + el.GDATE + '\n'
      })
      const anchor = document.createElement('a')
      anchor.href = 'data:text/csv;charset=utf-8,' + encodeURIComponent(csv)
      anchor.target = '_blank'
      anchor.download = 'DATA_LIST_' + new Date().toISOString().substr(0, 10) + '.csv'
      anchor.click()
    },
    dialogOpen () {
      this.inIsbn13 = ''
      this.dialog = true
    },
    insertData () {
      this.insertBook()
      window.location.href = 'http://localhost:3000/book?id=' + this.inIsbn13
    },
    async insertBook () {
      if (!this.inIsbn13) {
        window.alert('ISBN13が未設定です！' + this.inIsbn13)
        return 400
      }
      this.dialog = false
      try {
        // await this.$axios.$get('http://localhost:3000/api/bookinsert', {
        await this.$axios.$get('/api/bookinsert', {
          params: {
            id: this.inIsbn13
          }
        })
      } catch (e) {
        console.log(e.errorCode) // eslint-disable-line no-console
        window.alert(e)
        return e
      }
    },
    updateData () {
      window.location.href = 'http://localhost:3000/book?id=' + this.inIsbn13
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
.toc-view {
  background-color: #e6f2ff;
  border-radius: 13px;
  padding: 1rem;
  margin-left: 1rem;
  position: sticky;
  top: 1rem;
  max-height: 75vh;
  overflow: scroll;
}
</style>
