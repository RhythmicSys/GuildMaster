<template>
<div class="adventurer-section">
  <section class="recruit">
    <h1>Applying adventurers</h1>
    <div class="adventurers">
      <div v-if="currentlyForHire">
        <adventurer-tile class="hire-tile" :adventurer="currentlyForHire"/>
        <div class="decision">
          <span
              title="Hire"
              @click="hireAdventurer(currentlyForHire)"
              :class="{disabled: Object.keys(adventurers).length >= guild.adventurerCapacity.getAdventurerCapacity()}"
          >
            ✔
          </span>
          <span
              :title="Object.keys(adventurers).length > 0 ? 'Dismiss' : ''"
              :class="{disabled: Object.keys(adventurers).length <= 0}"
              @click="dismissAdventurer()"
          >
            ✗
          </span>
        </div>
      </div>
      <div v-else>
        <span>Noone applied as of now. Check back later!</span>
      </div>

    </div>
  </section>
  <section class="collection">
    <h1>Recruited adventurers ({{ Object.keys(adventurers).length }} / {{ guild.adventurerCapacity.getAdventurerCapacity() }})</h1>
    <div class="adventurers">
      <div class="adventurer-tile" v-for="adventurer in adventurers" :key="adventurer.id">
        <AdventurerTile class="entry" :adventurer="adventurer" />
        <b>{{ adventurer.name }}</b>
      </div>
    </div>
  </section>
</div>
</template>

<script lang="ts">
import type {PropType} from "vue";
import {defineComponent} from "vue";
import AdventurerTile from "@/components/AdventurerTile.vue";
import type {Adventurer} from "@/classes/Adventurer";
import { loadAdventurersForHire } from "@/GameData";
import type {Guild} from "@/classes/Guild";

export default defineComponent({
  name: "RecruitView",
  components: {AdventurerTile},
  data: () => {
    return {
      currentlyForHire: null as Adventurer|null,
      adventurersForHire: [] as Array<Adventurer>,
    }
  },
  props: {
    guild: {
      type: Object as PropType<Guild>,
      default() {
        return {} as Guild
      },
    },
    adventurers: {
      type: Object as PropType<{ [key: string]: Adventurer }>,
      default() {
        return {} as { [key: string]: Adventurer };
      },
    },
    lastRecruitTime: {
      type: Number as PropType<number>,
      default() {
        return 0;
      }
    },
  },
  methods: {
    getRandomAdventurer(): Adventurer|null {
      if (this.adventurersForHire.length <= 0) return null;
      const randomId = this.adventurersForHire.length * Math.random() << 0;
      return this.adventurersForHire[randomId];
    },
    getNewAdventurerForHire(): void {
      const adventurer = this.getRandomAdventurer();
      if (adventurer === null) {
        this.currentlyForHire = null;
        return;
      }

      if (this.adventurers[adventurer.id] != null) {
        this.currentlyForHire = null;
        return;
      }

      this.currentlyForHire = adventurer;
      window.localStorage.setItem("currentlyForHire", adventurer.id);

    },
    hireAdventurer(adventurer: Adventurer|any): void {
      if (Object.keys(this.adventurers).length >= this.guild.adventurerCapacity.getAdventurerCapacity()) return;
      this.adventurers[adventurer.id] = adventurer;
      this.currentlyForHire = null;
      window.localStorage.removeItem("currentlyForHire");
      this.$emit("recruitActionTaken", adventurer);
    },
    dismissAdventurer() {
      if (Object.keys(this.adventurers).length <= 0) return;
      this.currentlyForHire = null;
      this.$emit("recruitActionTaken", null);
      window.localStorage.removeItem("currentlyForHire");
    }
  },
  async mounted() {

    this.adventurersForHire = await loadAdventurersForHire(Object.keys(this.adventurers));

    if (Object.keys(this.adventurers).length <= 0) {
      this.currentlyForHire = this.adventurersForHire[0];
      window.localStorage.setItem("currentlyForHire", this.adventurersForHire[0].id);
      return;
    }

    const savedCurrentForHire = window.localStorage.getItem("currentlyForHire");
    if (savedCurrentForHire !== null) {
      for (const id in this.adventurersForHire) {
        const adventurer = this.adventurersForHire[id];
        if (adventurer.id === savedCurrentForHire) {
          this.currentlyForHire = adventurer;
          return;
        }
      }
    }

    const now = Number(new Date());

    if (now - this.lastRecruitTime >= 1000 * 60 * 30 && this.currentlyForHire === null) {
      this.getNewAdventurerForHire();
    }

  },
  emits: ["recruitActionTaken"]

});
</script>

<style lang="scss" scoped>
.adventurer-section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  section {
    max-width: 1280px;
    width: 100%;
    text-align: center;
    padding-block: 1rem;
  }
  h1 {
    font-size: 2rem;
    font-weight: bold;
    margin: 0;
  }
  .adventurers {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
    gap: 1rem;
    .adventurer-tile {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      gap: 0.25rem;
      font-size: 1.1rem;
      .entry {
        height: 7rem;
        width: 7rem;
      }
      b {
        line-height: 1;
        text-align: center;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
      }
    }
  }
  .decision {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
    gap: 1rem;
    span {
      cursor: pointer;
      &:hover {
        color: #fff;
      }
      &.disabled {
        color: rgba(0,0,0, 0.5);
        cursor: default;
      }
    }
  }
  .hire-tile {
    width: 8rem;
    height: 8rem;
  }

}
</style>
