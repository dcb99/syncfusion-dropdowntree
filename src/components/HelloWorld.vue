<template>
  <v-container>
    <v-row class="mx-auto">
      <br />
      <ejs-dropdowntree
        id="dropdowntree"
        :fields="dropdownFields"
        :allowFiltering="true"
        width="500"
        placeholder="Select Location"
      />
    </v-row>
  </v-container>
</template>

<script lang="ts">
import Vue from 'vue'
import { DropDownTreePlugin } from '@syncfusion/ej2-vue-dropdowns'
import * as locations from './../../location.json'

interface TreeData {
  nodeId: string;
  nodeText: string;
  nodeChild: TreeData[];
}

Vue.use(DropDownTreePlugin)

export default Vue.extend({
  data: () => ({
    treeItems: [] as TreeData[]
  }),
  computed: {
    dropdownFields () {
      return { dataSource: this.treeItems, value: 'nodeId', text: 'nodeText', child: 'nodeChild' }
    }
  },
  beforeMount () {
    this.parseData()
  },
  methods: {
    parseData () {
      const treeData = new Map<string, TreeData>()
      const topLevelParentIds = new Set<string>()

      locations.member.forEach(item => {
        if (item.parent === '' || item.parent === null) {
          topLevelParentIds.add(item.location)
        }

        treeData.set(item.location,
          {
            nodeId: item.location,
            nodeText: `${item.location}: ${item.description}`,
            nodeChild: []
          })
      })

      locations.member.forEach(item => {
        // eslint-disable-next-line
        const child = treeData.get(item.location)!
        // eslint-disable-next-line
        const parent = treeData.get(item.parent)!

        if (parent) {
          parent.nodeChild.push(child)
        }
      })

      const valuesAsArr = [...treeData.values()]
        .filter(x => topLevelParentIds.has(x.nodeId))
        .sort(this.compare)

      this.treeItems = valuesAsArr
    },
    compare (a: TreeData, b: TreeData) {
      return a.nodeChild.length < b.nodeChild.length
        ? 1
        : a.nodeChild.length > b.nodeChild.length
          ? -1
          : 0
    }
  }
})
</script>
