<template>
  <div class="data-table">
    <div class="form-group d-flex">
      <input
        v-model="searchValue" 
        type="text" 
        placeholder="Поиск по полям"
      />

      <button 
        @click="searchInTableFields" 
        class="btn btn-info btn-sm"
      >Найти</button>

      <button
        @click="isEdit = true"
        class="btn btn-primary btn-sm"
        :disabled="isEdit"
      >Добавить</button>

      <button
        @click="addHandler" 
        class="btn btn-success btn-sm"
        v-if="isFieldValid"
      >Добавить в таблицу</button>
    </div>

    <span v-if="copyData.length">Сортировка: {{ sortedBy }}</span>
    
    <div v-if="!copyData.length">Данные отсутствуют</div>

    <!-- Таблица клиентов -->
    <transition name="fade" mode="out-in">
      <table class="table table-sm table-hover mt-3" v-if="copyData.length">
        <thead>
          <tr>
            <th scope="col" @click="filterBy('id')">id</th>
            <th scope="col" @click="filterBy('firstName')">firstName</th>
            <th scope="col" @click="filterBy('lastName')">lastName</th>
            <th scope="col" @click="filterBy('email')">email</th>
            <th scope="col" @click="filterBy('phone')">phone</th>  
          </tr>
        </thead>
        <tbody>
          <tr v-if="isEdit">
            <th scope="col">
              <input v-model="addItem.id" v-mask="'######'"/>
            </th>
            <th scope="col">
              <input v-model="addItem.firstName" type="text" />
            </th>
            <th scope="col">
              <input v-model="addItem.lastName" type="text" />
            </th>
            <th scope="col">
              <input v-model="addItem.email" type="text" />
            </th>
            <th scope="col">
              <input v-model="addItem.phone" type="text" v-mask="'(###)###-####'" />
            </th>  
          </tr>
          <tr
            v-for="item in paginateItems" 
            :key="`${item.id}_${item.phone}`"
            @click="selectedItem = item"
          >
            <th scope="row">{{ item.id }}</th>
            <td>{{ item.firstName }}</td>
            <td>{{ item.lastName }}</td>
            <td>{{ item.email }}</td>
            <td>{{ item.phone }}</td>
          </tr>
        </tbody>
      </table>
    </transition>

    <!-- Пагинация таблицы -->
    <nav v-if="pages.length > 1">
      <ul class="pagination pagination-sm">
        <li 
          class="page-item" 
          v-if="page != 1" 
          @click="page--"
        >
          <span class="page-link">&laquo;</span>
        </li>
        <li 
          class="page-item" 
          :class="{active: page === pageNumber}" 
          :key="pageNumber" 
          v-for="pageNumber in pages.slice(page-1, page+5)" 
          @click="page = pageNumber"
        >
          <span class="page-link">{{ pageNumber }}</span>
        </li>
        <li 
          class="page-item" 
          @click="page++" 
          v-if="page < pages.length"
        >
          <span class="page-link">&raquo;</span>
        </li>
      </ul>
    </nav>

    <!-- Дополнительная информация о клиенте -->
    <item-info :selectedItem="selectedItem" />

  </div>
</template>

<script>

  import ItemInfo from "./ItemInfo";

  export default {
    name: "DataTable",
    components: {
      ItemInfo
    },
    props: {
      tableData: {
        type: Array,
        required: true
      }
    },
    data() {
      return {
        sortedBy: "отсутствует",
        selectedItem: null,
        page: 1,
        perPage: 50,
        pages: [],
        searchValue: "",
        copyData: [],
        isEdit: false,
        addItem: {
          id: null,
          firstName: null,
          lastName: null,
          email: null,
          phone: null
        },
        sortedUp: false,
        regEmail: /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      }
    },
    mounted() {
      this.setPages(this.tableData);
      this.copyData = this.tableData.slice();
    },
    methods: {
      filterBy(param) {

        this.sortedUp = !this.sortedUp;

        this.sortedUp
          ? this.filterByUp(param)
          : this.filterByDown(param)
      },

      filterByUp(param) {

        this.sortedBy = "по возрастанию"
        
        this.copyData = this.copyData.sort((a, b) => {
          var x = a[param]; 
          var y = b[param];

          return ((x < y) ? -1 : ((x > y) ? 1 : 0))
        });
      },

      filterByDown(param) {

        this.sortedBy = "по убыванию"
        
        this.copyData = this.copyData.sort((a, b) => {
          var x = a[param]; 
          var y = b[param];

          return ((x > y) ? -1 : ((x < y) ? 1 : 0))
        });
      },

      addHandler() {

        let item = Object.assign({}, this.addItem)

        this.copyData.unshift(item);
        this.addItem.id = null;
        this.addItem.id = null;
        this.addItem.firstName = null;
        this.addItem.lastName = null;
        this.addItem.email = null;
        this.addItem.phone = null;

        this.isEdit = false;
      },

      setPages (tableData) {
        let numberOfPages = Math.ceil(tableData.length / this.perPage);
        this.pages = [];
        for (let index = 1; index <= numberOfPages; index++) {
          this.pages.push(index);
        }
      },

      paginate (tableData) {
        let page = this.page;
        let perPage = this.perPage;
        let from = (page * perPage) - perPage;
        let to = (page * perPage);
        return tableData.slice(from, to);
      },

      searchInTableFields() {
        this.page = 1;
        this.isEdit = false;
        this.copyData = this.tableData.slice();
        this.copyData = this.tableData.filter(item => {
          return Object.values(item).some(field => {
            return String(field).toLowerCase().includes(this.searchValue.toLowerCase())
          })
        })
        this.setPages(this.copyData);
      }
    },
    computed: {
      isFieldValid() { 

        if(
          !this.regEmail.test(this.addItem.email) || 
          this.addItem.phone?.length < 13 ||
          /[0-9]/.test(this.addItem.firstName) ||
          /[0-9]/.test(this.addItem.lastName)
        ) {
          return false
        }

        return Object.values(this.addItem).every(field => field);
      },

      paginateItems () {
        return this.paginate(this.copyData);
      }
    },
  }
</script>

<style scoped>
  tr {
    cursor: pointer;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>