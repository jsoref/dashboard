<script>
import ResourceTable from '@/components/ResourceTable';
import Link from '@/components/formatter/Link';
import Shortened from '@/components/formatter/Shortened';

import {
  AGE,
  STATE,
  NAME,
  FLEET_SUMMARY
} from '@/config/table-headers';

export default {

  components: {
    ResourceTable, Link, Shortened
  },
  props: {
    rows: {
      type:     Array,
      required: true,
    },

    schema: {
      type:     Object,
      required: true,
    },
  },

  computed: {
    headers() {
      const out = [
        STATE,
        NAME,
        {
          name:     'repo',
          labelKey: 'tableHeaders.repo',
          value:    'repoDisplay',
          sort:     'repoDisplay',
          search:   ['spec.repo', 'status.commit'],
        },
        {
          name:     'target',
          labelKey: 'tableHeaders.target',
          value:    'targetInfo.modeDisplay',
          sort:     ['targetInfo.modeDisplay', 'targetInfo.cluster', 'targetInfo.clusterGroup'],
        },
        {
          name:      'clustersReady',
          labelKey:  'tableHeaders.clustersReady',
          value:     'status.readyClusters',
          sort:      'status.readyClusters',
          search:    false,
        },
        FLEET_SUMMARY,
        AGE
      ];

      return out;
    },
  },
  methods: {
    parseTargetMode(row) {
      return row.targetInfo?.mode === 'clusterGroup' ? this.t('fleet.gitRepo.warningTooltip.clusterGroup') : this.t('fleet.gitRepo.warningTooltip.cluster');
    }
  },
};
</script>

<template>
  <ResourceTable
    v-bind="$attrs"
    :schema="schema"
    :headers="headers"
    :rows="rows"
    key-field="_key"
    v-on="$listeners"
  >
    <template #cell:repo="{row}">
      <Link
        :row="row"
        :value="row.spec.repo"
        label-key="repoDisplay"
        before-icon-key="repoIcon"
        url-key="spec.repo"
      />
      <template v-if="row.commitDisplay">
        <div class="text-muted">
          <Shortened long-value-key="status.commit" :row="row" :value="row.commitDisplay" />
        </div>
      </template>
    </template>

    <template #cell:clustersReady="{row}">
      <span v-if="!row.clusterInfo" class="text-muted">&mdash;</span>
      <span v-else-if="row.clusterInfo.unready" class="text-warning">{{ row.clusterInfo.ready }}/{{ row.clusterInfo.total }}</span>
      <span v-else class="cluster-count-info">
        {{ row.clusterInfo.ready }}/{{ row.clusterInfo.total }}
        <i
          v-if="!row.clusterInfo.total"
          v-tooltip.bottom="parseTargetMode(row)"
          class="icon icon-warning"
        />
      </span>
    </template>

    <template #cell:target="{row}">
      {{ row.targetInfo.modeDisplay }}
    </template>
  </ResourceTable>
</template>

<style lang="scss">
.cluster-count-info {
  display: flex;
  align-items: center;

  i {
    margin-left: 5px;
    font-size: 22px;
    color: var(--warning);
  }
}
</style>
