<template>
    <div class="editions-dashboard mw-basic-layout">
        <div class="mw-content">
            <app-title :edition="edition"
                       :is-loading="isLoading"
                       :title="$t('EDITIONS_DASHBOARD.LABEL')">
                <template slot="back">
                    <app-back :to="back"></app-back>
                </template>
            </app-title>

            <md-tabs class="md-elevation-2"
                     md-alignment="fixed"
                     v-if="!isLoading">
                <md-tab :md-disabled="Object.keys(partners).length === 0"
                        :md-label="$tc('EDITIONS_DASHBOARD.PARTNERS.LABEL', Object.keys(partners).length, [Object.keys(partners).length])"
                        id="tab-partners"
                        md-icon="business_center">

                    <md-list class="md-triple-line"
                             v-if="Object.keys(partners).length > 0">
                        <md-list-item :key="`partner_${id}`"
                                      v-for="(partner, id) in partners">
                            <md-avatar>
                                <img :src="partner.logoUrl"
                                     alt=""/>
                            </md-avatar>

                            <div class="md-list-item-text">
                                <span>{{ partner.name }}</span>
                                <span>
                                    <a :href="`https://${partner.url}`"
                                       target="_blank">
                                        {{ partner.url }}
                                    </a>
                                    &nbsp;-&nbsp;
                                    {{ partner.level }}
                                </span>
                                <span>{{ $t('PARTNER.ACTIVE_ON') }} : {{ partner.activeOn | date('DD/MM/YYYY') }} ({{ partner.activeOn | date('v') }})</span>
                            </div>

                            <md-menu>
                                <md-button class="md-icon-button"
                                           md-menu-trigger>
                                    <md-icon>more_vert</md-icon>
                                </md-button>

                                <md-menu-content>
                                    <md-menu-item
                                            :to="{ name: 'partners-edit', params: { editionId: $route.params.editionId, partnerId: id } }">
                                        {{ $t('ACTIONS.MODIFY') }}
                                    </md-menu-item>
                                </md-menu-content>
                            </md-menu>
                        </md-list-item>
                    </md-list>

                </md-tab>

                <md-tab :md-disabled="Object.keys(speakers).length === 0"
                        :md-label="$tc('EDITIONS_DASHBOARD.SPEAKERS.LABEL', Object.keys(speakers).length, [Object.keys(speakers).length])"
                        id="tab-speakers"
                        md-icon="record_voice_over">

                    <md-list class="md-double-line"
                             v-if="Object.keys(speakers).length > 0">
                        <md-list-item :key="`speaker_${id}`"
                                      v-for="(speaker, id) in speakers">
                            <md-avatar>
                                <img :src="speaker.photoURL"
                                     alt=""/>
                            </md-avatar>

                            <div class="md-list-item-text">
                                <span>{{ speaker.displayName }}</span>
                                <span>{{ speaker.company }}</span>
                            </div>

                            <md-menu>
                                <md-button class="md-icon-button"
                                           md-menu-trigger>
                                    <md-icon>more_vert</md-icon>
                                </md-button>

                                <md-menu-content>
                                    <md-menu-item
                                            :to="{ name: 'speakers-edit', params: { editionId: $route.params.editionId, speakerId: id } }">
                                        {{ $t('ACTIONS.MODIFY') }}
                                    </md-menu-item>
                                </md-menu-content>
                            </md-menu>
                        </md-list-item>
                    </md-list>

                </md-tab>

                <md-tab :md-disabled="Object.keys(talks).length === 0"
                        :md-label="$tc('EDITIONS_DASHBOARD.TALKS.LABEL', Object.keys(talks).length, [Object.keys(talks).length])"
                        id="tab-talks"
                        md-icon="event">

                    <md-list v-if="Object.keys(talks).length > 0">
                        <md-list-item :key="`talk_${id}`"
                                      v-for="(talk, id) in talks">
                            <div class="md-list-item-text">{{ talk.title }}</div>

                            <md-menu>
                                <md-button class="md-icon-button"
                                           md-menu-trigger>
                                    <md-icon>more_vert</md-icon>
                                </md-button>

                                <md-menu-content>
                                    <md-menu-item
                                            :to="{ name: 'talks-edit', params: { editionId: $route.params.editionId, talkId: id } }">
                                        {{ $t('ACTIONS.MODIFY') }}
                                    </md-menu-item>
                                </md-menu-content>
                            </md-menu>
                        </md-list-item>
                    </md-list>

                </md-tab>
            </md-tabs>
        </div>

        <md-speed-dial class="cms-edit-btn">
            <md-speed-dial-target>
                <md-icon>add</md-icon>
            </md-speed-dial-target>

            <md-speed-dial-content>
                <md-button @click="add('editions-import')"
                           class="md-icon-button">
                    <md-icon>save</md-icon>
                     <md-tooltip md-direction="left">{{ $t('ACTIONS.VALID') }}
                    </md-tooltip>
                </md-button>
                <md-button @click="add('talks-edit')"
                           class="md-icon-button">
                    <md-icon>event</md-icon>
                    <md-tooltip md-direction="left">{{ $t('EDITIONS_DASHBOARD.TALKS.CREATE') }}
                    </md-tooltip>
                </md-button>
                <md-button @click="add('speakers-edit')"
                           class="md-icon-button">
                    <md-icon>record_voice_over</md-icon>
                     <md-tooltip md-direction="left">{{ $t('EDITIONS_DASHBOARD.SPEAKERS.CREATE') }}
                    </md-tooltip>
                </md-button>
                <md-button @click="add('partners-edit')"
                           class="md-icon-button">
                    <md-icon>business_center</md-icon>
                    <md-tooltip md-direction="left">{{ $t('EDITIONS_DASHBOARD.PARTNERS.CREATE') }}
                    </md-tooltip>
                </md-button>
            </md-speed-dial-content>
        </md-speed-dial>
    </div>
</template>

<script>
import AppBack from '@/components/app-back/AppBack.vue';
import AppTitle from '@/components/app-title/AppTitle';
import EditionsService from '@/services/EditionsService';
import PartnersService from '@/services/PartnersService';
import SpeakersService from '@/services/SpeakersService';
import TalksService from '@/services/TalksService';

export default {
  name: 'editions-dashboard',
  components: { AppBack, AppTitle },
  data() {
    return {
      isLoading: false,
      edition: {},
      partners: {},
      speakers: {},
      talks: {},
      back: { name: 'editions' },
    };
  },
  beforeRouteEnter(to, from, next) {
    EditionsService.findOneForCurrentUser(to.params.editionId)
      .then(edition => next(vm => vm.edition = edition))
      .catch(() => next({ name: 'editions' }));
  },
  created() {
    this.getDatas();
  },
  methods: {
    getDatas() {
      this.isLoading = true;

      Promise
        .all([
          PartnersService.findAllForEdition(this.$route.params.editionId),
          SpeakersService.findAllForEdition(this.$route.params.editionId),
          TalksService.findAllForEdition(this.$route.params.editionId),
        ])
        .then(datas => {
          this.partners = datas[0];
          this.speakers = datas[1];
          this.talks = datas[2];
          this.isLoading = false;
        })
        .catch(err => console.error(err));
    },
    add(name) {
      this.$router.push({ name: name, params: { editionId: this.$route.params.editionId } });
    },
  },
};
</script>
