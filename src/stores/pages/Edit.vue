<script>
import { connect } from 'vuex-connect'
import datastore from '@/base/datastore'
import StoreEdit from '@/stores/components/StoreEdit'

export default connect({
  stateToProps: {
    status: () => {
      const active = datastore.getters['stores/activeStore']
      return active && active.saveStatus
    },
  },
  gettersToProps: {
    value: 'stores/activeStore',
    allStores: 'stores/byCurrentGroup',
  },
  actionsToEvents: {
    save: 'stores/save',
  },
})('StoreEdit', StoreEdit)
</script>
