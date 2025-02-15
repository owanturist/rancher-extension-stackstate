<script>
import { mapGetters } from 'vuex';
import HealthState from '../components/Health/HealthState';
import { loadSuseObservabilitySettings, loadComponent, isCrdLoaded } from '../modules/suseObservability';
import { buildUrn } from '../modules/urn';
import { HEALTH_STATE_TYPES } from '../types/types';

export default {
  name:       'ComponentLinkedHealthState',
  components: { HealthState },
  props:      {
    value: {
      type:    String,
      default: '',
    },
    row: {
      type:     Object,
      required: true,
    },
  },

  computed: {
    ...mapGetters(['currentCluster']),

    componentIdentifier() {
      const cluster = this.currentCluster?.spec.displayName;

      return buildUrn(this.row, cluster);
    },

    color() {
      switch (this.value) {
      case 'active':
        return 'green';
      case 'inactive':
        return 'grey';
      default:
        return '';
      }
    },
  },
  data() {
    return {
      HEALTH_STATE_TYPES,
      isLoading: true,
      error:     null,
      data:      null
    };
  },

  async fetch() {
    if (!isCrdLoaded(this.$store)) {
      return;
    }

    const componentIdentifier = this.componentIdentifier;

    if (!componentIdentifier) {
      this.isLoading = false;

      return;
    }

    try {
      const settings = await loadSuseObservabilitySettings(this.$store);

      const component = await loadComponent(
        this.$store,
        settings,
        componentIdentifier
      );

      this.data = {
        health: component.state.healthState,
        href:   `https://${ settings.spec.url }/#/components/${ encodeURIComponent(
          componentIdentifier
        ) }`
      };
    } catch (error) {
      this.error = error;
    } finally {
      this.isLoading = false;
    }
  },
};
</script>

<template>
  <HealthState v-if="isLoading" :color="color" />

  <a
    v-else-if="data"
    :href="data.href"
    target="_blank"
    rel="nofollow noopener noreferrer"
  >
    <HealthState :health="data.health" :color="color" />
  </a>

  <HealthState v-else :health="HEALTH_STATE_TYPES.UNKNOWN" :color="color" />
</template>
