<template>
  <section>
    <section class="bigCard">
      <section>
        <label for="name">Name:</label>
        <input type="text" id="name" v-model="character.name">
      </section>
      <section>
        <label for="level">Level:</label>
        <input type="number" id="level" v-model.number="character.level">
      </section>
      <section>
        <label for="coreAttribute">Core Attribute:</label>
        <select id="coreAttribute" v-model="character.coreAttribute">
          <option value="Str">Str</option>
          <option value="Dex">Dex</option>
          <option value="Wis">Wis</option>
          <option value="Int">Int</option>
          <option value="Cha">Cha</option>
        </select>
      </section>
    </section>
    <section v-for="(stat, name) in counters" :key="name">
      <section class="outerCard">
        <label>{{ name }}:</label>
        <button @click="increment(name)">+</button>
        <span>{{ stat }}</span>
        <button @click="decrement(name)">-</button>
        <section class="card" v-if="(stat > 0)">
          {{ counterEffects[name].replace('X', stat) }}
        </section>
      </section>
    </section>
    <section>
      <h2>Attack Bonus: {{ calculateAttackBonus() }}</h2>
    </section>
    <section>
      <section v-for="(value, name) in conditions" :key="name">
        <section class="outerCard">
          <label>{{ name }}:</label>
          <input type="checkbox" v-model="conditions[name]">
          <br/>
          <section class="card" v-if="value">
            <h3>{{ name }}</h3>
            <p>{{ conditionEffects[name] }}</p>
            <p>Value: {{ value }}</p>
          </section>
        </section>
      </section>
    </section>
    <section>
      <h2>Conditions:</h2>
      <section v-for="(value, name) in conditions" :key="name"></section>
    </section>
  </section>
</template>

<script>
export default {
  data() {
    return {
      character: {
        name: '',
        level: 1,
        coreAttribute: 'Str'
      },
      counters: {
        Sickened: 0,
        Fatigued: 0,
        Frightened: 0,
        Enfeebled: 0,
        Clumsy: 0,
        Stupefied: 0,
        LifeDrain: 0,
        Corruption: 0,
        Dying: 0,
        Injured: 0
      },
      counterEffects: {
        Sickened: "Cannot willingly ingest anything AND -X to all d20 rolls.",
        Fatigued: "-X to AC and Saves.",
        Frightened: "-X to all d20 rolls.",
        Enfeebled: "-X to Str bonus.",
        Clumsy: "-X to Dex bonus.",
        Stupefied: "-X to Int bonus.",
        Befuddled: "-X to Wis bonus.",
        Apathetic: "-X to Cha bonus.",
        LifeDrain: "-X to Max HP.",
        Corruption: "-X Hit Dice.",
        Dying: "X steps toward death. Can only do one thing, big or small, each turn.",
        Injured: "If you begin Dying, the Dying value starts at X."
      },
      conditions: {
        Blind: false,
        Deaf: false,
        Slowed: false,
        Stunned: false,
        OffGuard: false,
        Prone: false,
        Entangled: false,
        Grabbed: false,
        Grappled: false,
        Pinned: false,
        Locked: false,
        Strangled: false,
        Fascinated: false,
        Fleeing: false,
        Confused: false,
        Quickened: false
      },
      conditionEffects: {
        Blind: 'Cannot perform actions dependent on sight, -10 to Perception',
        Deaf: 'Cannot perform actions dependent on hearing, -4 to Perception',
        Slowed: 'One enemy gets an extra turn before yours',
        Stunned: 'Your next action gives the enemy a turn token whether it succeeds or not',
        OffGuard: '-2 to AC',
        Prone: 'Off-Guard and have trouble moving',
        Entangled: 'Upgrade difficult terrain and -4 on actions requiring large movements',
        Grabbed: 'Cannot move away from the grabber until you escape, and -4 to checks with the grabbed limb',
        Grappled: 'Cannot move away from the grabber and cannot use the grabbed limb',
        Pinned: 'Cannot Move and have -4 on everything except grappling with the pinner',
        Locked: 'Cannot Move and have -4 to everything',
        Strangled: 'Cannot breathe or speak',
        Fascinated: 'Ignore all but the target of your fascination, walking slowly toward it in amazement',
        Fleeing: 'Must flee the source of your fear; recover by changing the situation (e.g. get away from the source, the source dies or is severely injured by an ally, sectionine intervention, a friend snaps you out of it, etc.)',
        Confused: 'Roll a die to determine how you act each turn',
        Quickened: 'Go before non-quickened creatures'
      }
    };
  },
  methods: {
    increment(stat) {
      this.counters[stat]++;
    },
    decrement(stat) {
      if (this.counters[stat] > 0) {
        this.counters[stat]--;
      }
    },
    calculateAttackBonus() {
      let bonus = this.character.level;
      
      return bonus + ' + Proficiency + ' + this.character.coreAttribute;
    },
    showCondition(name) {
      return this.conditions[name] || this.counters[name] > 0;
    }
  }
};
</script>

<style scoped>
  .card {
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-top: 10px;
    padding: 10px;
    text-align: center;
    background-color: #f9f9f9;
  }

  .outerCard {
    padding: 10px;
    padding-top: 3px;
    border-bottom: 1px solid #ccc;
  }

  .bigCard {
    text-align: left;
    margin-left: 100px;
    margin-top: 20px;
  }

  input[type="text"],
  input[type="number"],
  select {
    padding: 5px;
    margin-bottom: 10px;
    width: 100%;
    border-radius: 5px;
    border: 1px solid #ccc;
  }

  input[type="checkbox"] {
    margin-right: 5px;
  }

  label {
    display: inline-block;
    width: 100px;
    font-weight: bold;
  }

  button {
    padding: 5px
  }
</style>