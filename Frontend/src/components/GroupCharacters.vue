<template>
  <div class="flex flex-row flex-wrap group-characters">
    <box-character
      v-for="(object, index) in objects"
      :key="index"
      :object="object"
      :showLevel="showLevel"
      :showRing="showRing"
      @click-portrait="showModal(index)"
      @click-skill="clickSkill(index, $event)"
      @drag-portrait="drag($event, index)"
      @swap="swap($event, index)"
    ></box-character>

    <!-- Modal -->
    <modal
      v-if="party_mode !== $MODE.ReadOnly"
      v-model="show_modal"
      route="/party/characters"
      :categories="getCategories"
      @item-selected="changeObject"
    ></modal>    
  </div>
</template>

<script>
import { mapState } from 'vuex'

import Utils from '@/js/utils'

import BoxCharacter from '@/components/BoxCharacter.vue'
import Modal from '@/components/ModalSelector.vue'

const CATEGORIES = [
  {
    name: "Name",
    isColumn: true,
    isFilter: false,
    key: "n",
  },
  {
    name: "Rarity",
    isColumn: false,
    isFilter: true,
    key: "ri",
  },
  {
    name: "Element",
    isColumn: true,
    isFilter: true,
    key: "e",
  },
  {
    name: "Type",
    isColumn: true,
    isFilter: true,
    key: "t",
  },
  {
    name: "Race",
    isColumn: true,
    isFilter: true,
    key: "ra",        
  },
  {
    name: "Weapon",
    isColumn: true,
    isFilter: true,
    key: "w",        
  },
];

export default {
  components: {
    BoxCharacter,
    Modal,
  },
  props: {
    showLevel: {
      type: Boolean,
      default: false,
    },
    showRing: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      show_modal: false,
      selected_box_index: 0,
    }
  },
  methods: {
    drag(ev, index) {
      ev.dataTransfer.setData("character", JSON.stringify(index));
    },
    showModal(index) {
      switch (this.party_mode) {
        case this.$MODE.Action:
          // TODO show warning
          break;

        case this.$MODE.Edit:
          this.selected_box_index = index;
          this.show_modal = true;
          break;

        case this.$MODE.ReadOnly:
          // Do nothing
          break;
      }
    },
    changeObject(id) {
      const slot = this.selected_box_index;
      if (Utils.isEmpty(id)) {
        this.$store.commit('setCharacter', { index: slot, data: {} });
      }
      else {
        this.axios.get('/party/characters/' + id)
          .then(response => this.$store.commit('setCharacter', { index: slot, data: response.data }))
          .catch(error => this.$store.dispatch('addAxiosErrorMessage', error));
      }
    },
    clickSkill(index, skillIndex) {
      switch (this.party_mode) {
        case this.$MODE.Action:
          if (this.objects[index].skills[skillIndex] !== undefined) {
            this.$store.commit('addActionCharacterSkill', { slot: index, index: skillIndex });
          }
          break;

        case this.$MODE.Edit:
          // TODO warning
          break;

        case this.$MODE.ReadOnly:
          // Do nothing
          break;
      }
    },
    swap(from, to) {
      if (this.party_mode === this.$MODE.ReadOnly) {
        return;
      }
      let tmp = this.objects[from];
      this.$store.commit('setCharacter', { index: from, data: this.objects[to] })
      this.$store.commit('setCharacter', { index: to, data: tmp })
    }
  },
  computed: {
    ...mapState({
      objects: state => state.party_builder.characters,
      party_mode: state => state.party_builder.party_mode
    }),
    getCategories() {
      return CATEGORIES;
    }
  }
}
</script>

<style scoped>
.group-characters > :nth-child(3) {
  @apply mr-2;
}
</style>