<template>
  <vue-crud-x ref="categories" storeName="categories" :parentId="null" v-bind="categoryDefs"></vue-crud-x>
</template>

<script>
import VueCrudX from '../../src/VueCrudX'

export default {
  middleware: ['auth'],
  name: 'categories',
  components: {
    VueCrudX
  },
  data() {
    return {
      categoryDefs: {
        crudTable: {
          actionColumn: false,
          addrowCreate: false,
          // inline: false,
          confirmCreate: true,
          confirmUpdate: true,
          confirmDelete: true,
          headers: [{ text: 'Category Name', value: 'name', class: 'pa-1' }],
          formatters: (value, _type) => value,
          doPage: 2
        },
        crudFilter: {
          FilterVue: null,
          filterData: {}
        },
        crudForm: {
          FormVue: null,
          formAutoData: {
            id: { type: 'input', attrs: { hidden: true } }, // need id if there is delete
            name: {
              type: 'v-text-field',
              attrs: {
                label: 'Name',
                rules: [v => !!v || 'Item is required']
              },
              halfSize: false
            }
          },
          defaultRec: () => ({
            id: '',
            name: ''
          })
        },
        crudOps: {
          export: async payload => {},
          find: async payload => {
            let records = []
            let totalRecords = 0
            const { pagination } = payload // filterData
            const { page, rowsPerPage } = pagination // sortBy, descending
            try {
              const {
                data: { results, total }
              } = await this.$axios.get('/api/categories', {
                params: {
                  page: page > 0 ? page - 1 : 0,
                  limit: rowsPerPage
                }
              })
              records = results
              totalRecords = total
            } catch (e) {
              console.log(e)
            }
            return { records, pagination, totalRecords }
          },
          findOne: async payload => {
            const { id } = payload
            try {
              const { data } = await this.$axios.get(`/api/categories/${id}`)
              return data
            } catch (e) {}
            return {}
          },
          create: async payload => {
            try {
              let {
                record: { id, ...noIdData }
              } = payload
              const rv = await this.$axios.post('/api/categories', noIdData)
              console.log(rv)
            } catch (e) {
              return 500
            }
            return 201
          },
          update: async payload => {
            try {
              let {
                record: { id, ...noIdData }
              } = payload
              const rv = await this.$axios.patch(`/api/categories/${id}`, noIdData)
              console.log(rv)
            } catch (e) {
              if (parseInt(e.message) === 409) return 409
              else return 500
            }
            return 200
          },
          delete: null // TBD if delete, must also delete all dependancies, move all buttons to right?
        }
      }
    }
  },
  methods: {}
}
</script>
