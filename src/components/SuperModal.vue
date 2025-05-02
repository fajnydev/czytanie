<template>
  <div class="modal">
    <div class="box">
      <button class="close" @click="$emit('close')">✕</button>
      <h2>Super moce</h2>

      <section class="mode-buttons">
        <button @click="$emit('set-mode','letters')">Przejdź do liter</button>
        <button @click="$emit('set-mode','syllables')">Przejdź do sylab</button>
        <button @click="$emit('set-mode','words')">Przejdź do wyrazów</button>
      </section>

      <fieldset class="checker">
        <legend>Usuń błędne hasła</legend>
        <input v-model="customText" placeholder="Wpisz słowo / sylabę"/>
        <div class="row">
          <button @click="checkReading">Sprawdź czytanie</button>
          <button @click="addToBlacklist" :disabled="!customText">Usuń z programu</button>
        </div>
      </fieldset>

      <section v-if="blacklist.length" class="blacklist">
        <h3>Wyłączone słowa</h3>
        <ul>
          <li v-for="w in blacklist" :key="w">
            {{ w }}
            <button @click="$emit('restore',w)">Przywróć</button>
          </li>
        </ul>
      </section>

      <section v-if="progressLetters.length || progressSyllables.length" class="progress">
        <h3>Postępy</h3>

        <div class="tableWrap" v-if="progressLetters.length">
          <h4>Litery</h4>
          <table>
            <tr>
              <th>Litera</th>
              <th>✓</th>
              <th>✗</th>
              <th>Śr. ms</th>
            </tr>
            <tr v-for="row in progressLetters" :key="'l'+row.text">
              <td>{{ row.text }}</td>
              <td>{{ row.correct }}</td>
              <td>{{ row.wrong }}</td>
              <td>{{ row.avgMs }}</td>
            </tr>
          </table>
        </div>

        <div class="tableWrap" v-if="progressSyllables.length">
          <h4>Sylaby</h4>
          <table>
            <tr>
              <th>Sylaba</th>
              <th>✓</th>
              <th>✗</th>
              <th>Śr. ms</th>
            </tr>
            <tr v-for="row in progressSyllables" :key="'s'+row.text">
              <td>{{ row.text }}</td>
              <td>{{ row.correct }}</td>
              <td>{{ row.wrong }}</td>
              <td>{{ row.avgMs }}</td>
            </tr>
          </table>
        </div>
      </section>
    </div>
  </div>
</template>
<script>
export default {
  name: 'SuperModal',
  props: {blacklist: Array, progressLetters: Array, progressSyllables: Array},
  data: () => ({customText: ''}),
  methods: {
    checkReading() {
      if (!this.customText) return;
      const u = new SpeechSynthesisUtterance(this.customText);
      u.lang = 'pl-PL';
      u.rate = 0.9;
      window.speechSynthesis.speak(u);
    },
    addToBlacklist() {
      this.$emit('blacklist', this.customText.trim().toLowerCase());
      this.customText = '';
    }
  }
}
</script>
<style scoped>
.modal {
  position: fixed;
  inset: 0;
  background: #0008;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 20
}

.box {
  background: #fff;
  padding: 2rem;
  border-radius: 12px;
  max-width: 340px;
  width: 92%;
  position: relative;
  text-align: center
}

.close {
  position: absolute;
  top: 6px;
  right: 10px;
  border: none;
  background: none;
  font-size: 1.4rem;
  cursor: pointer
}

.mode-buttons button, .row button {
  display: block;
  width: 100%;
  margin: .5rem 0;
  padding: .6rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer
}

.mode-buttons button {
  background: #1a8cff;
  color: #fff
}

.checker {
  border: 2px dashed #1a8cff;
  border-radius: 10px;
  padding: 1rem;
  margin-top: 1rem
}

.checker legend {
  font-weight: 600;
  padding: 0 .4rem
}

.checker input {
  width: 100%;
  padding: .6rem;
  margin: .4rem 0;
  border: 1px solid #ccc;
  border-radius: 6px;
  box-sizing: border-box;
}

.row {
  display: flex;
  gap: .5rem
}

.row button:first-child {
  background: #4caf50;
  color: #fff
}

.row button:last-child {
  background: #f44336;
  color: #fff
}

.blacklist h3 {
  margin-top: 1rem;
  font-size: 1rem
}

.blacklist ul {
  list-style: none;
  padding: 0;
  max-height: 140px;
  overflow: auto;
  margin: 0
}

.blacklist li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: .2rem 0;
  border-bottom: 1px solid #eee
}

.blacklist li button {
  background: #ff9800;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: .2rem .6rem;
  font-size: .8rem;
  cursor: pointer
}

.progress h3 {
  margin: 1.2rem 0 .6rem;
  font-size: 1rem
}

.tableWrap {
  overflow: auto;
  max-height: 180px;
  margin-bottom: .5rem
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: .9rem
}

th, td {
  padding: .2rem .4rem;
  border-bottom: 1px solid #ddd;
  text-align: center
}

td:first-child {
  text-align: left
}

</style>
