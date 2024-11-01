<template>
  <div class="full-width-wrapper">
    <div class="beholder-container">
      <div class="content">
        <div class="left-column">
          <div class="ac-hp-stats">
            <div class="ac-hp-row">
              <h2 class="ac-label">AC: {{ beholder.ac }}</h2>
              <h2 class="hp-label">HP: {{ beholder.hp }} / {{ maxHp }}</h2>
              <div class="hp-controls">
                <div>
                  <button @click="updateHP(true)">+</button>
                  <input type="number" v-model="hpChange" placeholder="Change HP" />
                  <button @click="updateHP(false)">-</button>
                </div>
              </div>
              <button @click="startNewRound">Start New Round</button>
            </div>
          </div>

          <div class="stats-lair-actions">
            <div class="stats-saves">
              <h2>Stats and Saves</h2>
              <table>
                <thead>
                <tr>
                  <th>Stat</th>
                  <th>Modifier</th>
                  <th>Save</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="(stat, key) in beholder.stats" :key="key">
                  <td>{{ key }}</td>
                  <td>{{ calculateModifier(stat.value) }}</td>
                  <td>{{ stat.save }}</td>
                </tr>
                </tbody>
              </table>
            </div>

            <div style="padding: 1rem;">
              <div class="legendary-resistances">
                <h2>Legendary Resistances</h2>
                <div v-for="(index) in beholder.legendaryResistances" :key="index">
                  <label>
                    <input type="checkbox" @change="(event) => { if (event.target.checked) { event.target.disabled = true; } }" />
                    Resistance {{ index }}
                  </label>
                </div>
              </div>

              <div class="legendary-actions">
                <h2>Legendary Actions</h2>
                <div v-for="(action, index) in beholder.legendaryActions" :key="index">
                  <label>
                    <input type="checkbox" v-model="action.used" :disabled="!canUseLegendaryAction" />
                    {{ action.name }}: {{ action.description }}
                  </label>
                </div>
              </div>
            </div>
          </div>

          <div class="actions-lair-actions">
            <h2>Actions</h2>
            <div class="actions">
              <div v-for="(action, index) in beholder.actions" :key="index" class="action">
                <strong>{{ action.name }}</strong>: {{ action.description }}
              </div>
            </div>

            <h2>Lair Actions</h2>
            <p>
              On initiative count 20, the beholder can take one of the following lair actions. The same lair action cannot be used two rounds in a row.
            </p>
            <table>
              <thead>
              <tr>
                <th>Action</th>
                <th>Description</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="action in beholder.lairActions" :key="action.name">
                <td>{{ action.name }}</td>
                <td>{{ action.description }}</td>
              </tr>
              </tbody>
            </table>
          </div>

          <div class="inventory">
            <h2>Inventory</h2>
            <table>
              <thead>
              <tr>
                <th>Item</th>
                <th>Description</th>
                <th>Charges</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="(item, index) in beholder.inventory" :key="index" class="item">
                <td>
                  <label>
                    <input type="checkbox"
                           :checked="item.equipped"
                           @change="toggleItemEffect(item)" />
                    <strong>{{ item.name }}</strong>
                  </label>
                </td>
                <td>{{ item.description }}</td>
                <td>
                  <div v-if="item.charges > 0">
                    <label v-for="n in item.charges" :key="n">
                      <input type="checkbox" @change="(event) => { if (event.target.checked) { event.target.disabled = true; } }" />
                    </label>
                  </div>
                </td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>

        <div class="right-column">
          <div class="eye-rays">
            <table>
              <thead>
              <tr>
                <th>Roll</th>
                <th>Ray</th>
                <th>Description</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="ray in beholder.eyeRays" :key="ray.roll">
                <td>{{ ray.roll }}</td>
                <td>{{ ray.name }}</td>
                <td>{{ ray.description }}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    beholder: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      maxHp: this.beholder.hp, // Store max HP
      hpChange: 0 // For updating HP
    };
  },
  created() {
    this.applyInitialItemEffects(); // Apply effects from initially equipped items
  },
  methods: {
    applyInitialItemEffects() {
      this.beholder.inventory.forEach(item => {
        if (item.equipped) {
          this.applyItemEffect(item, true); // Apply effects as equipping
        }
      });
    },
    toggleItemEffect (item) {
      // Toggle equipped status
      item.equipped = !item.equipped;
      this.applyItemEffect(item, item.equipped); // Apply effects based on the new equipped status
    },
    applyItemEffect (item, isEquipping) {
      if (item.effect) { // Check if the item has an effect
        const { type, adjustment } = item.effect;

        if (type === 'adjustStats') {
          // Adjust AC if defined
          if (adjustment.ac) {
            this.beholder.ac += isEquipping ? adjustment.ac : -adjustment.ac; // Adjust AC correctly
          }

          // Adjust saving throws if applicable
          if (adjustment.savingThrows) {
            for (const [saveStat, value] of Object.entries(adjustment.savingThrows)) {
              this.beholder.stats[saveStat].save += isEquipping ? value : -value; // Adjust saving throws correctly
            }
          }
        }
      }
    },
    calculateModifier (value) {
      return Math.floor((value - 10) / 2); // Calculate the modifier
    },
    startNewRound() {
      // Reset legendary actions
      this.beholder.legendaryActions.forEach(action => {
        action.used = false;
      });

      console.log('Current HP:', this.beholder.hp);
      console.log('Max HP:', this.maxHp);

      // Check if the beholder's HP is less than max HP before applying regeneration effects
      if (this.beholder.hp < this.maxHp) {
        console.log('Applying regeneration effects');
        // Apply regeneration effects
        this.beholder.inventory.forEach(item => {
          if (item.equipped && item.effect && item.effect.type === 'regenerate') {
            const regenAmount = this.calculateDiceRoll(item.effect.amount); // Calculate the 2d6 HP
            console.log('Regeneration Amount:', regenAmount);

            this.beholder.hp = Math.min(this.beholder.hp + regenAmount, this.maxHp); // Set HP to max if exceeded
            console.log('New HP:', this.beholder.hp);
          }
        });
      }
    },
    calculateDiceRoll (dice) {
      const [numDice, dieType] = dice.split('d').map(Number);
      let total = 0;
      for (let i = 0; i < numDice; i++) {
        total += Math.floor(Math.random() * dieType) + 1; // Roll the die
      }
      return total;
    },
    updateHP (isPositive) {
      const change = isPositive ? this.hpChange : -this.hpChange; // Determine change
      this.beholder.hp += change; // Directly add or subtract from current HP
      this.hpChange = 0; // Reset input
    },
  },
  computed: {
    canUseLegendaryAction() {
      //get count of legendary actions that have been used
      const usedActions = this.beholder.legendaryActions.filter(action => action.used).length;
      return usedActions < this.beholder.legendaryActionCount;

    }
  }
}
</script>

<style scoped>
.ac-hp-stats {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.ac-hp-row {
  display: flex;
  justify-content: center; /* Center the items */
  align-items: center; /* Center vertically */
  margin-bottom: 1rem; /* Space below the row */
  gap: 1rem; /* Space between items */
}

.hp-controls {
  display: flex; /* Add flexbox to align items */
  align-items: center; /* Center the buttons and input vertically */
  gap: 0.5rem; /* Space between buttons and input */
  margin-top: 0; /* Remove top margin to align with labels */
}

.ac-label,
.hp-label {
  margin: 0; /* Reset margin for consistent height */
}

.full-width-wrapper {
  width: 95vw;
  overflow: hidden;
}

.beholder-container {
  padding: 1rem;
  box-sizing: border-box;
}

.content {
  display: flex;
  flex-wrap: wrap;
  width: 100%;
}

.left-column,
.right-column {
  flex: 1;
  margin: 0 10px;
}

.stats-lair-actions {
  display: flex;
  justify-content: space-between; /* Space between stats and LA */
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 0.5rem;
  border: 1px solid #ccc;
  text-align: left;
}

th {
  color: white;
}
</style>

