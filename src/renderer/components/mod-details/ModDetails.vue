<!-- eslint-disable vue/no-v-html -->
<template>
  <v-card
    tile
    class="d-flex"
    width="100%"
    height="100%"
    style="padding: 0; box-shadow: inset 10px 0 10px -10px rgba(0,0,0,1); max-width: calc(100vw - 550px);"
  >
    <v-overlay
      v-if="!mod || mod.mod_reference !== expandedModId"
      absolute
    >
      Loading...
    </v-overlay>
    <ModDetailsInfo
      v-if="mod"
      :mod="mod"
      :is-installed="isInstalled"
      @close="close"
      @install-version="installVersion"
    />
    <v-card
      v-if="mod"
      height="100%"
      width="100%"
      class="mod-description-card"
    >
      <v-container
        fluid
        class="d-flex flex-column"
        style="height: 100%; padding: 20px 12x 15px 20px;"
      >
        <v-row style="flex-basis: 0; overflow: auto;">
          <div
            class="mod-description"
            v-html="modDescription"
          />
        </v-row>
        <v-row style="flex-grow: 0;">
          <v-spacer />
          <v-col style="flex-grow: 0; padding-bottom: 0">
            <v-btn
              raised
              width="150px"
              @click="favoriteClicked"
            >
              <span style="vertical-align: middle;">
                {{ isFavorite ? 'Unfavourite' : 'Favourite' }}
              </span>
              <v-spacer />
              <v-icon
                right
                :color="isFavorite ? 'warning' : 'text'"
              >
                mdi-star
              </v-icon>
            </v-btn>
          </v-col>
          <v-col
            v-if="isCompatible || isInstalled"
            style="flex-grow: 0; padding-bottom: 0"
          >
            <v-btn
              width="150px"
              raised
              :disabled="!!inProgress.length || !canInstallMods"
              @click="switchClicked"
            >
              <span style="vertical-align: middle;">
                {{ isInstalled ? 'Remove mod' : 'Install mod' }}
              </span>
              <v-spacer />
              <v-icon
                right
                :color="isInstalled ? 'red' : 'text'"
              >
                {{ isInstalled ? 'mdi-delete' : 'mdi-download' }}
              </v-icon>
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </v-card>
</template>

<script>
import { mapState, mapGetters } from 'vuex';
import gql from 'graphql-tag';
import { markdownAsElement, isCompatibleFast } from '@/utils';
import ModDetailsInfo from './ModDetailsInfo';

export default {
  components: {
    ModDetailsInfo,
  },
  data() {
    return {
    };
  },
  computed: {
    ...mapState([
      'inProgress',
      'expandedModId',
    ]),
    ...mapGetters([
      'canInstallMods',
    ]),
    isFavorite() {
      return this.$store.state.favoriteModIds.includes(this.expandedModId);
    },
    descriptionAsElement() {
      return markdownAsElement(this.mod.full_description || '');
    },
    modDescription() {
      const el = this.descriptionAsElement;
      const links = el.getElementsByTagName('a');
      for (let i = 0; i < links.length; i += 1) {
        links[i].target = '_blank';
      }
      return el.innerHTML;
    },
    isInstalled() {
      return !!this.$store.state.installedMods[this.expandedModId];
    },
  },
  asyncComputed: {
    isCompatible: {
      get() {
        if (!this.$store.state.selectedInstall) return false;
        return isCompatibleFast(this.mod, this.$store.state.selectedInstall.version);
      },
      default: false,
    },
  },
  apollo: {
    mod: {
      query: gql`
        query getModDetails($modReference: ModReference!) {
          mod: getModByReference(modReference: $modReference)
          {
            id,
            name,
            logo,
            mod_reference,
            full_description,
            created_at,
            last_version_date,
            downloads,
            views,
            hidden,
            authors {
              user {
                username,
                avatar
              }
              role
            },
            versions(filter: {limit: 100}) {
              id,
              version,
              sml_version,
              size
            }
          }
        }
      `,
      variables() {
        return {
          modReference: this.expandedModId,
        };
      },
      pollInterval: 5 * 60 * 1000,
    },
  },
  methods: {
    close() {
      this.$store.dispatch('unexpandMod');
    },
    switchClicked() {
      this.$store.dispatch('switchModInstalled', this.expandedModId);
    },
    favoriteClicked() {
      this.$store.dispatch('toggleModFavorite', this.expandedModId);
    },
    installVersion(version) {
      this.$store.dispatch('installModVersion', { modId: this.expandedModId, version });
    },
  },
};
</script>

<style>
.mod-description img {
  max-width: 100%;
}
.mod-description {
  font-size: 13.5px;
}
.v-application .mod-description p {
  margin-bottom: 5px;
}
</style>

<style scoped>
.v-icon {
  font-size: 20px !important;
}
.v-card {
  background-color: var(--v-backgroundModsList-base) !important;
}
.v-btn {
  color: var(--v-text-base) !important;
  font-size: 16px !important;
  padding-left: 8px !important;
  padding-right: 12px !important;
}

.mod-description {
  display: block;
  word-break: break-word;
  overflow-y: auto;
  box-shadow: none;
}
.mod-description-card {
  max-width: calc(100% - 296px);
}
@media (max-height: 700px) {
  .mod-description-card {
    max-width: calc(100% - 260px);
  }
}

.row {
  margin-left: 0 !important;
  margin-right: 0 !important;
}
</style>
