<template>
  <div class="hello">
    <b-row>
    <b-col lg="6" class="my-1">
      <b-form-group
        label="Filter"
        label-cols-sm="3"
        label-align-sm="right"
        label-size="sm"
        label-for="filterInput"
        class="mb-0"
      >
        <b-input-group size="sm">
          <b-form-input
            v-model="filter"
            type="search"
            id="filterInput"
            placeholder="Type to Search"
          ></b-form-input>
          <b-input-group-append>
            <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
          </b-input-group-append>
        </b-input-group>
      </b-form-group>
    </b-col>
    <b-col lg="6" class="my-1">
      <b-form-group
        label="Filter On"
        label-cols-sm="3"
        label-align-sm="right"
        label-size="sm"
        description="Leave all unchecked to filter on all data"
        class="mb-0">
        <b-form-checkbox-group v-model="filterOn" class="mt-1">
          <b-form-checkbox value="company">Company</b-form-checkbox>
          <b-form-checkbox value="city">City</b-form-checkbox>
          <b-form-checkbox value="country">Country</b-form-checkbox>
          <b-form-checkbox value="countryCode">Country Code</b-form-checkbox>
          <b-form-checkbox value="currencyCode">Currency Code</b-form-checkbox>
        </b-form-checkbox-group>
      </b-form-group>
    </b-col>

    <b-col sm="5" md="6" class="my-1">
        <b-form-group
          label="Per page"
          label-cols-sm="6"
          label-cols-md="4"
          label-cols-lg="3"
          label-align-sm="right"
          label-size="sm"
          label-for="perPageSelect"
          class="mb-0"
        >
          <b-form-select
            v-model="perPage"
            id="perPageSelect"
            size="sm"
            :options="pageOptions"
          ></b-form-select>
        </b-form-group>
      </b-col>

      <b-col sm="7" md="6" class="my-1">
        <b-pagination
          v-model="currentPage"
          :total-rows="totalRows"
          :per-page="perPage"
          align="fill"
          size="sm"
          class="my-0"
        ></b-pagination>
      </b-col>
    </b-row>
    <b-form-group label="Selection mode:" label-cols-md="4">
      <b-form-select v-model="selectMode" :options="modes" class="mb-3"></b-form-select>
    </b-form-group>
    <b-table 
    ref="selectableTable"
    selectable
    :select-mode="selectMode"
    @row-selected="onRowSelected"
    show-empty
    stacked="md"
    :items="data" 
    :fields="fields" 
    :current-page="currentPage"
    :per-page="perPage"
    :filter="filter"
    :filter-included-fields="filterOn"
    @filtered="onFiltered"
    striped responsive="sm">
    <template v-slot:cell(edit)="row">
        <b-button @click="edit(row.item.company)">Edit</b-button>
    </template>
    <template v-slot:cell(selected)="{ rowSelected }">
        <template v-if="rowSelected">
          <span aria-hidden="true">&check;</span>
          <span class="sr-only">Selected</span>
        </template>
        <template v-else>
          <span aria-hidden="true">&nbsp;</span>
          <span class="sr-only">Not selected</span>
        </template>
      </template>
    </b-table>
    <p>
      <b-button size="sm" @click="selectAllRows">Select all</b-button>
      <b-button size="sm" @click="clearSelected">Clear selected</b-button>
      <b-button size="sm" @click="del">Delete Selected</b-button>
    </p>
    <p>
      Selected Rows:<br>
      {{ selected }}
    </p>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'HelloWorld',
  props: {
    msg: String,
    error_: []
  },
  data() {
    return {
      api: "http://192.168.43.158:3000",
      data: [],
      filter: null,
      serach: '',
      // fields: ['#', 'Company', 'City', 'Country', 'Country Code', 'Currency Code']
      fields: [
        'selected',
        { key: 'id', label: '#' },
        { key: 'company', label: 'Company' },
        { key: 'city', label: 'City', filterByFormatted: true },
        { key: 'country', label: 'Country' },
        { key: 'countryCode', label: 'Country Code' },
        { key: 'currencyCode', label: 'Currency Code' },
        { key: 'edit', label: '' }
      ],
      filterOn: [],
      currentPage: 1,
      perPage: 100,
      totalRows: 1,
      pageOptions: [25, 50, 100],
      modes: ['multi', 'single'],
      selectMode: 'multi',
      selected: [],
      selectedCompany: []
    }
  },
  created() {
    this.dtb()
  },
  mounted() {
    this.totalRows = this.data.length
  },
  methods: {
    dtb() {
      axios.get(this.api).then(res => {
        this.data = res.data
        console.log(res.data)
      }).catch(e => {
        this.error_.push(e)
      })
      console.log(this.error_)
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
    onRowSelected(items) {
        this.selected = items
        // console.log(this.selected)
        // this.selectedCompany.push(this.selected[this.selected.length-1].company)
        // console.log(this.selectedCompany)
    },
    selectAllRows() {
      this.$refs.selectableTable.selectAllRows()
    },
    clearSelected() {
      this.$refs.selectableTable.clearSelected()
    },
    edit(company, city, country, countryCode, currencyCode) {
      var param = {
        company: company,
        city: city,
        countryCode: countryCode,
        currencyCode: currencyCode
      }
      axios.post(this.api + '/edit', param).then(res => {
        console.log(res)
      }).catch(e => {
        this.error_.push(e)
      })
      console.log(this.error_)
    },
    del() {
      // console.log(this.selected.length)
      // console.log(this.selected)
      for (let i = this.selected.length; i--; i >= 0) {
        this.selectedCompany.push(this.selected[i].company)
      }
      var param = {
        company: this.selectedCompany
      }
      console.log(param)
      axios.post(this.api + '/delete', param).then(res => {
        console.log(res)
      }).catch(e => {
        this.error_.push(e)
      })
      console.log(this.error_)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
