<script lang='tsx'>
import { reactive, computed, defineComponent } from 'vue'

const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1)

export default defineComponent({
  name: "GridDemo",
  props: {
    data: Array,
    columns: Array,
    filterKey: String
  },
  setup(props) {
    const state = reactive({
      sortKey: '',
      sortOrders: props.columns.reduce((o, key) => (o[`${key}`] = 1, o), {})
    })

    const filteredData = computed(() => {
      let { data, filterKey } = props
      if (filterKey) {
        filterKey = filterKey.toLowerCase()
        data = data.filter(row => {
          return Object.keys(row as {}).some(key => {
            return String(row[key]).toLowerCase().indexOf(filterKey) > -1
          })
        })
      }
      const { sortKey } = state
      if (sortKey) {
        const order = state.sortOrders[sortKey]
        data = data.slice().sort((a, b) => {
          a = a[sortKey]
          b = b[sortKey]
          return (a === b ? 0 : a > b ? 1 : -1) * order
        })
      }
      return data
    })

    function sortBy(key) {
      state.sortKey = key
      state.sortOrders[key] *= -1
    }
    return () => (
        <div>
          {
            filteredData.value.length ? (
              <table>
                <thead>
                  <tr>
                  {
                    props.columns.map(key => (
                      <th
                        onClick={() => sortBy(key)}
                        class={{active: state.sortKey === key}}>
                        { capitalize(key) }
                        <span class={{
                          'arrow': true,
                          'dsc': state.sortOrders[`${key}`] < 0,
                          'asc':  state.sortOrders[`${key}`] > 0
                        }}>
                      </span>
                    </th>
                    ))
                  }
                  </tr>
                </thead>
                <tbody>
                  {
                    filteredData.value.map(entry => (
                      <tr>
                        {
                          props.columns.map(key => (
                            <td>
                              {entry[`${key}`]}
                            </td>
                          ))
                        }
                        
                      </tr>
                    ))
                  }
                  
                </tbody>
              </table>
            ) : <p>No matches found.</p>
        }
      </div>
    )
  }
})
</script>
<style>
table {
  border: 2px solid #42b983;
  border-radius: 3px;
  background-color: #fff;
}

th {
  background-color: #42b983;
  color: rgba(255,255,255,0.66);
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

td {
  background-color: #f9f9f9;
}

th, td {
  min-width: 120px;
  padding: 10px 20px;
}

th.active {
  color: #fff;
}

th.active .arrow {
  opacity: 1;
}

.arrow {
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  margin-left: 5px;
  opacity: 0.66;
}

.arrow.asc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-bottom: 4px solid #fff;
}

.arrow.dsc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid #fff;
}
</style>
