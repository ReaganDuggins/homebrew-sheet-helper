<template>
  <section>
    <section class="bigCard">
      <button id="resetButton" @click="resetCharacter">Reset Character</button>
      <section class="baseStat">
        <label for="name">Name:</label>
        <input @input="saveForm" type="text" id="name" v-model="character.name">
      </section>
      <section class="baseStat">
        <label for="level">Level:</label>
        <input @input="saveForm" type="number" min="0" max="20" id="level" v-model.number="character.level">
      </section>
      <section class="baseStat">
        <label for="maxHP">Max HP:</label>
        <input @input="saveForm" type="number" min="0" id="maxHP" v-model.number="character.maxHP">
      </section>
      <section class="baseStat">
        <label for="dieAtDying">Die At Dying Value:</label>
        <input @input="saveForm" type="number" min="4" id="dieAtDying" v-model.number="character.dieAtDying">
      </section>
      <section class="baseStat">
        <label for="coreAttribute">Core Attribute:</label>
        <select @change="saveForm" id="coreAttribute" v-model="character.coreAttribute">
          <option value="Str">Str</option>
          <option value="Dex">Dex</option>
          <option value="Wis">Wis</option>
          <option value="Int">Int</option>
          <option value="Cha">Cha</option>
        </select>
      </section>
      <section class="baseStat">
        <label for="attributeBonus">Core Attribute Bonus:</label>
        <input @input="saveForm" type="number" min="-5" id="attribteBonus" v-model.number="character.attributeBonus">
        <section class="plusMinusButtons">
          <button class="plusMinusButton" @click="incrementCharacter('attributeBonus')">+</button>
          <button class="plusMinusButton" @click="decrementCharacter('attributeBonus')">-</button>
        </section>
      </section>
      <section class="baseStat">
        <label for="proficiency">Max Available Proficiency:</label>
        <select @change="saveForm" id="proficiency" v-model="character.proficiency">
          <option value="trained">Trained</option>
          <option value="expert">Expert</option>
          <option value="master">Master</option>
          <option value="legend">Legend</option>
          <option value="untrained">Untrained</option>
        </select>
      </section>
    </section>

    <section class="rownator">
      <h1>Attack Bonus</h1>
      <section>
        <b>The Equation:</b> Level + {{ character.coreAttribute }} + Proficiency Bonus - Penalties
      </section>
      <section>
        <b>The Bonus:</b> {{ totalAttackPenalty() }}
      </section>
      <h1>Conditions</h1>
      <section class="row">
        <section class="rowElement" v-for="condition in getConditionsByKind('num')" v-bind:key="condition.name">
          <section v-if="condition.kind === 'num'">
            <h3>
              {{ condition.name == 'Dying' ? (character.dying ? "Dying" : "Wounded") : condition.name }} 
              <span v-if="condition.name === 'Dying'">
                <button v-if="!character.dying" @click="toggleDyingWounded"><img class="icon" alt="Wound" src="./assets/skull-crossed-bones.png"></button>
                <button v-if="character.dying" @click="toggleDyingWounded"><img class="icon" alt="Wound" src="./assets/bleeding-wound.png"></button>
              </span>
            </h3>
            <section class="inputWithButtons">
              <input @input="saveForm" type="number" min="0" v-model="condition.value" />
              <section class="plusMinusButtons">
                <button class="plusMinusButton" @click="increment(condition)">+</button>
                <button class="plusMinusButton" @click="decrement(condition)">-</button>
              </section>
            </section>
          </section>
        </section>
      </section>

      <section class="row">
        <section class="rowElement" v-for="condition in getConditionsByKind('bool')" v-bind:key="condition.name">
          
          <section v-if="condition.kind === 'bool'">
            <button 
              @click="toggleCondition(condition.name)"
              :class="{ 'conditionButton': !condition.value, 'conditionButtonActive': condition.value }"
            >
              {{ condition.name }}
            </button>
          </section>
        </section>
      </section>

      <section class="row">
        <section class="cardDanger" v-if="conditions['Strangled'].value === true">
          <h4>DANGER</h4>
          <p>While <b>Strangled</b> You can hold your breath for up to 1 minute per Str Bonus. Vigorous movement expends 1 minute.</p><p>After that, you go unconscious on the next round.</p><p>If at the start of a round you are both Strangled and Unconscious, you start Dying.</p><p>If at the start of a round you are Dying and Strangled, you die.</p>
        </section>

        <!-- Death Card -->
        <section 
          v-if="conditions.Dying.value > 2"
        >
          <!-- dying -->
          <section v-if="character.dying">
            <section class='cardDeath' v-if="conditions.Dying.value >= character.dieAtDying">
              <h4>R.I.P.</h4>
              <p>You are dead... Your soul now sails down the River of Death...</p>
            </section>
            <section class='cardDanger' v-if="conditions.Dying.value < character.dieAtDying">
              {{ console.log(conditions.Dying.value, character.dieAtDying) }}
              <h4>DANGER</h4>
              <p>You have {{ character.dieAtDying - conditions.Dying.value }} rounds to live!</p>
            </section>
          </section>
          <!-- wounded -->
          <section v-if="!character.dying">
            <section class="cardDanger" v-if="conditions.Dying.value > 2">
              <section v-if="conditions.Dying.value >= character.dieAtDying">
                <h4>DANGER</h4>
                <p>You are in great danger! If you begin Dying, you will <b>DIE INSTANTLY!!!</b></p>
              </section>
              <section v-if="conditions.Dying.value < character.dieAtDying">
                <h4>DANGER</h4>
                <p>You are in great danger! If you begin Dying, you will only have {{ character.dieAtDying - conditions.Dying.value }} rounds to live!</p>
              </section>
            </section>
          </section>

        </section>

        <!-- Corruption Card -->
        <section :class="{'cardDanger': (character.level - conditions.Corruption.value) >= 0, 'cardDeath': (character.level - conditions.Corruption.value) < 0, }" v-if="conditions['Corruption'].value > Math.floor(character.level * 0.75)">
          <section v-if="(character.level - conditions.Corruption.value) >= 0">
            <h4>DANGER</h4>
            <p>You are {{ (character.level + 1) - conditions.Corruption.value  }} Corruption away from being instantly, permanently transformed by the source of the corruption.</p>
          </section>
          <section v-if="(character.level - conditions.Corruption.value) < 0">
            <h4>R.I.P.</h4>
            <p>You have been overcome by the source of your corruption. Rest in peace...</p>
          </section>
        </section>

        <section class="cardDanger" v-if="conditions['LifeDrain'].value > Math.floor(character.maxHP * 0.75)">
          <h4>DANGER</h4>
          <p>You are {{ character.maxHP - conditions['LifeDrain'].value }} Life Drain away from INSTANT DEATH!!!.</p>
        </section>
      </section>

      <section class="row">
        <section 
          v-for="condition, index in getActiveConditions()" 
          v-bind:key="condition" 
          v-bind:class="{ 'cardLight': index % 2 === 0, 'cardDark': index % 2 !== 0 }"
        >
          <h3>{{ condition.name }} <span v-if="condition.kind === 'num'">{{ condition.value }}</span></h3>
          <p>{{ condition.description.replace('X', condition.value) }}</p>

        </section>
      </section>

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
        maxHP: 10,
        dieAtDying: 4,
        attributeBonus: 0,
        coreAttribute: 'Str',
        proficiency: 'trained',
        dying: false
      },
      proficiencyValues: {
        'untrained': 0,
        'trained': 2,
        'expert': 4,
        'master': 6,
        'legend': 8
      },
      conditions: {
        Sickened: {
          value: 0, 
          kind: "num", 
          description: "Cannot willingly ingest anything AND -X to all d20 rolls."
        },
        Fatigued: {
          value: 0, 
          kind: "num", 
          description: "-X to AC and Saves."
        },
        Frightened: {
          value: 0, 
          kind: "num", 
          description: "-X to all d20 rolls."
        },
        Enfeebled: {
          value: 0, 
          kind: "num", 
          description: "-X to Str bonus."
        },
        Clumsy: {
          value: 0, 
          kind: "num", 
          description: "-X to Dex bonus."
        },
        Stupefied: {
          value: 0, 
          kind: "num", 
          description: "-X to Int bonus."
        },
        Befuddled: {
          value: 0, 
          kind: "num", 
          description: "-X to Wis bonus."
        },
        Apathetic: {
          value: 0, 
          kind: "num", 
          description: "-X to Cha bonus."
        },
        LifeDrain: {
          value: 0, 
          kind: "num", 
          description: "-X to Max HP."
        },
        Corruption: {
          value: 0, 
          kind: "num", 
          description: "-X Hit Dice."
        },
        Dying: {
          value: 0, 
          kind: "num", 
          description: "You are near death..."
        },
        Blind: {
          value: false, 
          kind: "bool", 
          description: "Cannot perform actions dependent on sight, -10 to Perception."
        },
        Deaf: {
          value: false, 
          kind: "bool", 
          description: "Cannot perform actions dependent on hearing, -4 to Perception."
        },
        Slowed: {
          value: false, 
          kind: "bool", 
          description: "You never go before non-Slowed enemies."
        },
        Stunned: {
          value: false, 
          kind: "bool", 
          description: "Your next action gives the enemy a turn token even if it succeeds."
        },
        OffGuard: {
          value: false, 
          kind: "bool", 
          description: "-2 to AC"
        },
        Prone: {
          value: false, 
          kind: "bool", 
          description: "Off-Guard and have trouble moving"
        },
        Entangled: {
          value: false, 
          kind: "bool", 
          description: "All speeds reduced to 5ft and all actions requiring large motions have -4."
        },
        Grabbed: {
          value: false, 
          kind: "bool", 
          description: "Cannot move away from the grabber until you escape, and -4 to checks with the grabbed limb"
        },
        Grappled: {
          value: false, 
          kind: "bool", 
          description: "Cannot move away from the grabber and cannot use the grabbed limb."
        },
        Pinned: {
          value: false, 
          kind: "bool", 
          description: "Cannot Move and have -4 on everything except grappling with the pinner."
        },
        Locked: {
          value: false, 
          kind: "bool", 
          description: "Cannot Move and have -4 to everything."
        },
        Strangled: {
          value: false, 
          kind: "bool", 
          description: "Cannot breathe or speak"
        },
        Fascinated: {
          value: false, 
          kind: "bool", 
          description: "Ignore all but the target of your fascination, walking slowly toward it in amazement"
        },
        Fleeing: {
          value: false, 
          kind: "bool", 
          description: "Must flee the source of your fear; recover by changing the situation (e.g. get away from the source, the source dies or is severely wounded by an ally, sectionine intervention, a friend snaps you out of it, etc.)"
        },
        Confused: {
          value: false, 
          kind: "bool", 
          description: "Roll a die to determine how you act each turn (ask the GM)."
        },
        Quickened: {
          value: false, 
          kind: "bool", 
          description: "Do an extra thing at the start of the round."
        },
      },
    };
  },
  methods: {
    toggleDyingWounded() {
      this.character.dying = !this.character.dying;
    },
    increment(stat) {
      this.conditions[stat.name]['value']++;
      this.saveForm();
    },
    decrement(stat) {
      if (this.conditions[stat.name]['value'] > 0) {
        this.conditions[stat.name]['value']--;
        this.saveForm();
      }
    },
    incrementCharacter(stat) {
      this.character[stat]++;
      this.saveForm();
    },
    decrementCharacter(stat) {
      if (this.character[stat] > -5) {
        this.character[stat]--;
        this.saveForm();
      }
    },
    calculateAttackBonus() {
      let bonus = this.character.level;
      
      return bonus + ' + Proficiency + ' + this.character.coreAttribute;
    },
    showCondition(name) {
      return this.conditions[name] || this.counters[name] > 0;
    },
    getActiveConditions() {
      let keys = Object.keys(this.conditions);
      let conds = keys.map((key) => {
        if(this.conditions[key].value) {
          let cond = this.conditions[key];
          cond['name'] = key;
          return cond;
        }
        return undefined;
      }).filter((cond) => {
        return !!cond;
      });
      return conds;
    },
    toggleCondition(name) {
      this.conditions[name].value = !this.conditions[name].value;
      this.saveForm();
    },
    getConditionsByKind(kind) {
      let keys = Object.keys(this.conditions);
      let conds = keys.map((key) => {
        if(typeof this.conditions[key].value === 'boolean') {
          this.conditions[key].kind = 'bool';
        }
        if(typeof this.conditions[key].value === 'number') {
          this.conditions[key].kind = 'num';
        }
        if(this.conditions[key].kind && this.conditions[key].kind.includes(kind)) {
          this.conditions[key]['name'] = key;
          return this.conditions[key];
        }
        return undefined;
      }).filter((cond) => {
        return !!cond;
      });
      return conds;
    },
    totalAttackPenalty() {
      let total = this.character.level + this.character.attributeBonus + this.proficiencyValues[this.character.proficiency]
      total -= this.conditions.Sickened.value;
      total -= this.conditions.Frightened.value;

      if(this.conditions.Entangled.value ||
        this.conditions.Grabbed.value || 
        this.conditions.Grappled.value ||
        this.conditions.Pinned.value ||
        this.conditions.Locked.value
      ) {
        total -= 4;
      }

      switch(this.character.coreAttribute) {
        case("Str"):
        total -= this.conditions.Enfeebled.value;
        break;

        case("Dex"):
        total -= this.conditions.Clumsy.value;
        break;

        case("Int"):
        total -= this.conditions.Stupefied.value;
        break;

        case("Wis"):
        total -= this.conditions.Befuddled.value;
        break;

        case("Cha"):
        total -= this.conditions.Apathetic.value;
        break;
      }

      return total;
    },
    saveForm() {
      let character = JSON.stringify(this.character);
      let conditions = JSON.stringify(this.conditions);
      localStorage.setItem('character', character);
      localStorage.setItem('conditions', conditions);
    },
    loadForm() {
      let character = localStorage.getItem('character');
      let conditions = localStorage.getItem('conditions');
      let char = JSON.parse(character);
      let cond = JSON.parse(conditions);
      if(!char || !cond) {
        return;
      }
      if(char) this.character = char;
      if(cond)  this.conditions = cond;
    },
    resetCharacter() {
      let conditions = Object.keys(this.conditions);
      let charStats = Object.keys(this.character);
      conditions.forEach((cond) => {
        if (this.conditions[cond].kind === "num") {
          this.conditions[cond].value = 0;
        }
        if (this.conditions[cond].kind === "bool") {
          this.conditions[cond].value = false;
        }
      });
      charStats.forEach((stat) => {
        if(typeof this.character[stat] === 'number') {
          this.character[stat] = 1;
        }
        if(typeof this.character[stat] === 'boolean') {
          this.character[stat] = false;
        }
      });
      this.character.name = "";
      charStats.coreAttribute = 'Str';
      charStats.proficiency = 'trained';
      this.saveForm();
    }
  },
  mounted() {
    this.loadForm();
  },
};
</script>

<style scoped>
  * {
    font-family: Helvetica, Arial, sans-serif;
  }

  .icon {
    width: 30px;
    height: 30px;
  }

  .cardLight {
    border: 1px solid black;
    border-radius: 5px;
    margin-top: 10px;
    padding-left: 1vw;
    padding-right: 1vw;
    padding-bottom: 2vw;
    text-align: center;
    background-color: #f9f9f9;
    width: 15vw;
  }

  .cardDark {
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-top: 10px;
    padding-left: 1vw;
    padding-right: 1vw;
    padding-bottom: 2vw;
    text-align: center;
    color: white;
    background-color: #6b6b6b;
    width: 15vw;
  }

  .cardDanger {
    border: 1px solid red;
    border-radius: 5px;
    margin-top: 10px;
    padding-left: 1vw;
    padding-right: 1vw;
    padding-bottom: 2vw;
    text-align: center;
    color: white;
    background-color: darkred;
    width: 30vw;
  }

  .cardDeath {
    border: 1px solid red;
    border-radius: 5px;
    margin-top: 10px;
    padding-left: 1vw;
    padding-right: 1vw;
    padding-bottom: 2vw;
    text-align: center;
    color: white;
    background-color: black;
    width: 30vw;
  }

  .conditionButton {
    border-radius: 10px;
    background-color: lightgreen;
    padding: 10px 30px;
    font-size:large;
    width: 150px
  }

  .conditionButtonActive {
    border-radius: 10px;
    background-color: red;
    padding: 10px 30px;
    font-size:large;
    width: 150px;
  }


  .outerCard {
    padding: 10px;
    padding-top: 3px;
    border-bottom: 1px solid #ccc;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .bigCard {
    text-align: left;
    margin-left: 30px;
    margin-right: 30px;
    margin-top: 20px;
    display: flex;
    border-bottom: 2px solid grey;
  }

  .baseStat {
    margin: 1vh 2vw;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
  }

  input[type="text"],
  input[type="number"],
  select {
    padding: 5px;
    margin-bottom: 10px;
    width: 100%;
    border-radius: 5px;
    width: 10vw;
    border: 1px solid #ccc;
  }
  
  input[type="number"]{
    height: 3vh;
    width: 4vw;
  }

  input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
  }

  input[type="text"] {
    width: 20vw;
    height: 3vh;
    margin: 2vh 2vw;
  }

input[type="checkbox"] {
  margin-left: 5px;
  width: 30px;
  height: 30px;
  border-radius: 5px;
}

label {
    font-weight: bold;
}

.card-description {
    margin-left: 20px;
    margin-right: 20px;
    flex: 1;
    color: #666;
}

.rownator {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  padding: 2vh 2vw;
  margin-left: 30px;
  margin-right: 30px;
  border-bottom: 2px solid grey;
}
.row {
  width: 90vw;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  padding-bottom: 5px;
}
.rowElement {
  margin: 5px 30px 5px 30px;
}

.colnator {
  display: flex;
  flex-direction: row;
  flex-wrap:wrap;
}
.col {
  width: 100vw;
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
}
.colElement {
  margin: 10px 30px 10px 30px;
}
.inputWithButtons {
  display: flex;
  justify-content: center;
  align-items: center;
}

.plusMinusButtons {
  display: flex;
  flex-direction: column;
  padding-bottom: 10px;
}
.plusMinusButton {
  background: blue;
  color: white;
  border: 1px groove grey;
  width: 50px;
  height: 22px;
}

h1 {
  font-size: xx-large;
  border-bottom: 1px solid black;
  width: 60vw;
  margin-bottom: 5px;
}
h3 {
  font-size: larger;
  border-bottom: 1px solid grey;
  width: 13w;
}
h4 {
  font-size: large;
  border-bottom: 1px solid grey;
  width: 13w;
}
</style>