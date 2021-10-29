<template>
  <v-card outlined>
    <div class="caption mb-n1 mt-n3">TALENT RANKS</div>
    <v-card flat>
      <v-tooltip
        v-for="(rank, i) in item.ranks"
        :key="`rank_chip_${item.id || item.name}-${i}`"
        top
      >
        <template v-slot:activator="{ on }">
          <v-chip
            large
            label
            close
            color="primary darken-3"
            class="ma-2"
            @click="edit(rank, i)"
            @click:close="remove(i)"
            v-on="on"
          >
            RANK {{ 'I'.repeat(i + 1) }} // {{ rank.name }}
          </v-chip>
        </template>
        <div v-html="rank.description" />
        <v-divider />
        <div v-if="rank.cost">Costs {{ rank.cost }} Uses</div>
        <div v-if="rank.exclusive">
          Exclusive:
          <b class="warning--text">true</b>
        </div>
      </v-tooltip>

      <v-dialog v-model="dialog">
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on"><v-icon>mdi-plus</v-icon></v-btn>
        </template>
        <v-card>
          <v-toolbar dense color="pink darken-4" class="text-h6">Add Rank</v-toolbar>
          <v-card-text>
            <v-row justify="space-around" align="center" class="mt-2">
              <v-col>
                <v-text-field label="Name" hide-details v-model="name" />
              </v-col>
              <v-col cols="auto">
                <v-tooltip top>
                  <template v-slot:activator="{ on }">
                    <v-switch v-model="exclusive" dense hide-details label="Exclusive" v-on="on" />
                  </template>
                  The exclusive field determines if only the bonuses, deployables, counters, and
                  integrated equipment from the highest unlocked rank will be equipped, or all
                  unlocked ranks. For example, the system granted by Walking Armory is upgraded
                  every rank. In C/C this is rendered by changing the granted system. Because the
                  player is equipped with the "latest" system, based on talent rank, the
                  talent_items in the Walking Armory ranks are all set to true.
                </v-tooltip>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <rich-text-editor title="Description" v-model="description" />
              </v-col>
            </v-row>
            <v-row align="center">
              <v-col><i-action-builder :item="this" /></v-col>
              <v-col><i-bonus-builder :item="this" /></v-col>
              <v-col><i-deployable-builder :item="this" /></v-col>
              <v-col><i-synergy-builder :item="this" /></v-col>
            </v-row>
            <v-row align="center">
              <v-col><i-counter-builder :item="this" /></v-col>
              <v-col><integrated-selector :item="this" /></v-col>
            </v-row>
          </v-card-text>
          <v-divider />
          <v-card-actions>
            <v-btn text color="error" @click="dialog = false">cancel</v-btn>
            <v-spacer />
            <v-btn color="success darken-2" :disabled="!confirmOK" @click="submit">
              {{ isEdit ? 'save' : 'confirm' }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-card>
  </v-card>
</template>

<script lang="ts">
import RichTextEditor from './RichTextEditor.vue'
import ISynergyBuilder from './ISynergyBuilder.vue'
import IActionBuilder from './IActionBuilder.vue'
import IBonusBuilder from './IBonusBuilder.vue'
import IDeployableBuilder from './IDeployableBuilder.vue'
import ICounterBuilder from './ICounterBuilder.vue'
import IntegratedSelector from './IntegratedSelector.vue'

import Vue from 'vue'
export default Vue.extend({
  name: 'rank-builder',
  props: { item: { type: Object, required: true } },
  components: {
    RichTextEditor,
    ISynergyBuilder,
    IActionBuilder,
    IBonusBuilder,
    IDeployableBuilder,
    ICounterBuilder,
    IntegratedSelector,
  },
  data: () => ({
    dialog: false,
    name: '',
    description: '',
    synergies: [],
    actions: [],
    bonuses: [],
    synergy_locations: [],
    deployables: [],
    counters: [],
    integrated: [],
    exclusive: false,
    isEdit: false,
    editIndex: -1,
  }),
  computed: {
    confirmOK(): boolean {
      return !!this.name && !!this.description
    },
  },
  methods: {
    submit(): void {
      const e = {
        name: this.name,
        description: this.description,
        synergies: this.synergies,
        actions: this.actions,
        bonuses: this.bonuses,
        synergy_locations: this.synergy_locations,
        deployables: this.deployables,
        counters: this.counters,
        integrated: this.integrated,
        exclusive: this.exclusive,
      }
      if (this.isEdit) {
        this.$set(this.item.ranks, this.editIndex, e)
      } else {
        if (!this.item.ranks) this.$set(this.item, 'ranks', [])
        this.item.ranks.push(e)
      }
      this.reset()
      this.dialog = false
    },
    edit(rank: any, index: number): void {
      this.name = rank.name
      this.description = rank.description
      this.synergies = rank.synergies
      this.actions = rank.actions
      this.bonuses = rank.bonuses
      this.synergy_locations = rank.synergy_locations
      this.deployables = rank.deployables
      this.counters = rank.counters
      this.integrated = rank.integrated
      this.exclusive = rank.exclusive
      this.isEdit = true
      this.editIndex = index
      this.dialog = true
    },
    remove(index: number): void {
      this.item.ranks.splice(index, 1)
    },
    reset(): void {
      this.name = ''
      this.description = ''
      this.synergies = []
      this.actions = []
      this.bonuses = []
      this.synergy_locations = []
      this.deployables = []
      this.counters = []
      this.integrated = []
      this.exclusive = false
      this.isEdit = false
    },
  },
})
</script>