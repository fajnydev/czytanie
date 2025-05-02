
<template><div class="reading-item"><span :class="['item',type]">{{ text }}</span></div></template>
<script>export default{name:'ReadingItem',props:{text:String,type:String}}</script>
<style scoped>
.reading-item{text-align:center;margin-bottom:1.5rem}
.item{font-size:5rem;font-weight:700}
.item.letter{color:#1a8cff}.item.syllable{color:#2e7d32}.item.word{color:#000}
@media(max-width:600px){.item{font-size:3.5rem}}
</style>
