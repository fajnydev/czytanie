
<template>
  <div class="controls">
    <button class="btn correct" @click="$emit('answered',true)">✅ Dobrze</button>
    <button class="btn incorrect" @click="$emit('answered',false)">❌ Źle</button>
  </div>
</template>
<script>export default{name:'Controls'}</script>
<style scoped>
.controls{display:flex;flex-wrap:wrap;gap:1rem;justify-content:center;width:100%}
.btn{flex:1 1 45%;font-size:1.8rem;padding:1rem;border:none;border-radius:10px;color:#fff;cursor:pointer}
.btn.correct{background:#4caf50}.btn.incorrect{background:#f44336}
.btn:active{transform:scale(.95)}
@media(min-width:601px){.btn{flex:0 0 200px}}
</style>
