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
    <!-- ダイアログ：表紙イメージ -->
    <v-dialog
      v-model="dialog2"
      max-width="350"
      persistent
    >
      <v-img
        max-width="350"
        :src="imgUrl"
      />
      <v-row no-gutters justify="center">
        <v-btn
          width="175"
          @click="updateData()"
        >
          編集
        </v-btn>
        <v-btn
          width="175"
          @click="dialog2 = false"
        >
          閉じる
        </v-btn>
      </v-row>
    </v-dialog>
    <v-row>
      <v-col>
        <v-card-title>
          BOOKLOG
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
          <v-btn
            class="success"
            @click="dialogOpen()"
          >
            登録
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
    >
      <!-- template #[`item.ISBN13`]="{ item }">
        <a :href="`http://localhost:3000/book?id=${item.ISBN13}`">
          {{ item.ISBN13 }}
        </a>
      </template -->
      <template #[`item.Purchase`]="{ item }">
        <div align="right">
          {{ item.Purchase }}
        </div>
      </template>
    </v-data-table>
  </div>
</template>

<script>
/* eslint-disable no-console */
export default {
  name: 'BooklogList',
  async asyncData ({ $axios }) {
    try {
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
      dialog2: false,
      inIsbn13: '',
      selRow: 0,
      headers: [
        {
          text: 'ISBN13',
          value: 'ISBN13'
        },
        {
          text: '書籍名',
          value: 'BookName'
        },
        {
          text: '著者',
          value: 'Author'
        },
        {
          text: '出版社',
          value: 'Publisher'
        },
        {
          text: '価格',
          value: 'Purchase',
          align: 'end',
          width: '90'
        },
        {
          text: '分類',
          value: 'Genre'
        },
        {
          text: '発行日',
          value: 'IDATE',
          width: '125'
        }
      ],
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
    rowClick (row) {
      this.inIsbn13 = row.ISBN13
      this.imgUrl = 'https://images-na.ssl-images-amazon.com/images/I/' + row.CoverImg
      this.selRow = this.lists.indexOf(row)
      this.dialog2 = true
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
</style>
