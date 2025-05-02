
<template>
  <div class="app">
    <button class="power-btn" @click="showModal=true">⚡</button>

    <transition name="fade" mode="out-in">
      <ReadingItem :key="currentItem.text" :text="currentItem.text" :type="currentItem.type"/>
    </transition>

    <Controls @answered="handleAnswer"/>

    <SuperModal v-if="showModal"
                :blacklist="blacklist"
                @close="showModal=false"
                @set-mode="setManualMode"
                @blacklist="addToBlacklist"
                @restore="restoreWord"/>
  </div>
</template>

<script>
import ReadingItem from './components/ReadingItem.vue';
import Controls from './components/Controls.vue';
import SuperModal from './components/SuperModal.vue';
import { lettersBase, syllablesBase, wordsBase } from './data/readingData.js';

export default{
  name:'App',
  components:{ ReadingItem, Controls, SuperModal },
  data(){
    return{
      lettersBase, syllablesBase, wordsBase,
      stage:'letters',
      manualMode:null,
      currentItem:{text:'',type:''},
      startTime:null,
      immediateQueue:[],
      syllableQueue:[], wordQueue:[],
      stats:{},
      delayId:null,
      showModal:false,
      blacklist: JSON.parse(localStorage.getItem('readingBlacklist')||'[]')
    };
  },
  created(){
    const saved=localStorage.getItem('readingStats');
    if(saved) this.stats=JSON.parse(saved);
    this.resetQueues();
    this.nextItem();
  },
  methods:{
    setManualMode(m){this.manualMode=m;this.showModal=false;this.nextItem();},
    addToBlacklist(w){if(!w)return;if(!this.blacklist.includes(w)){this.blacklist.push(w);}localStorage.setItem('readingBlacklist',JSON.stringify([...this.blacklist]));},
    restoreWord(w){this.blacklist = this.blacklist.filter(wo => wo !== w);localStorage.setItem('readingBlacklist',JSON.stringify([...this.blacklist]));},
    resetQueues(){this.syllableQueue=[...this.syllablesBase];this.wordQueue=this.wordsBase.map(w=>({...w}));},
    pickRandom(a){return a[Math.floor(Math.random()*a.length)]},
    pickWeightedLetter(){
      const stats=this.stats.letters??{};
      let total=0;
      const bucket=this.lettersBase.map(l=>{
        const c=stats[l]?.correct??0;
        const w=Math.max(1,5-c);total+=w;return [l,w];
      });
      let r=Math.random()*total;
      for(const [ltr,w] of bucket){if(r<w)return ltr;r-=w;}
      return this.lettersBase[0];
    },
    nextItem(){
      let item;
      const mode=this.manualMode||this.stage;
      do{
        if(this.immediateQueue.length){item=this.immediateQueue.shift();}
        else if(mode==='letters'){item={text:this.pickWeightedLetter(),type:'letter'};}
        else if(mode==='syllables'){item={text:this.pickRandom(this.syllableQueue),type:'syllable'};}
        else {item={...this.pickRandom(this.wordQueue),type:'word'};}
      }while(this.blacklist.includes(item.text.toLowerCase()))
      this.currentItem=item;this.startTime=Date.now();
    },
    enqueueImmediate(e){this.immediateQueue.push(e);},
    enqueueLater(e){
      if(e.type==='syllable') this.syllableQueue.push(e.text);
      else if(e.type==='word') this.wordQueue.push({text:e.text,syllables:e.syllables});
    },
    handleAnswer(ok){
      const item=this.currentItem;
      const t=Date.now()-this.startTime;
      this.saveStats(item,ok,t);
      this.speak(item.text);
      if(!ok){
        this.enqueueLater(item);
        if(item.type==='word'){
          item.syllables.forEach(s=>this.enqueueImmediate({text:s,type:'syllable'}));
          this.enqueueImmediate(item);
        }else if(item.type==='syllable'){
          item.text.split('').forEach(l=>this.enqueueImmediate({text:l,type:'letter'}));
          this.enqueueImmediate(item);
        }
      }else if(item.type!=='letter'){
        this.maybeAdvanceStage();
      }
      clearTimeout(this.delayId);
      this.delayId=setTimeout(()=>this.nextItem(),1000);
    },
    saveStats(item,ok,time){
      const cat=item.type+'s';
      this.stats[cat]??={};
      this.stats[cat][item.text]??={correct:0,wrong:0,total:0};
      const e=this.stats[cat][item.text];
      ok?e.correct++:e.wrong++; e.total+=time;
      localStorage.setItem('readingStats',JSON.stringify(this.stats));
    },
    maybeAdvanceStage(){
      if(this.stage==='letters'){
        const mastered=Object.values(this.stats.letters??{}).filter(e=>e.correct>=3&&e.wrong<=e.correct).length;
        if(mastered>=this.lettersBase.length*0.7) this.stage='syllables';
      }else if(this.stage==='syllables'){
        const m=Object.values(this.stats.syllables??{}).filter(e=>e.correct>=3&&e.wrong<=e.correct).length;
        if(m>=this.syllablesBase.length*0.6) this.stage='words';
      }
    },
    speak(txt){
      if(!('speechSynthesis'in window))return;
      const pl=speechSynthesis.getVoices().find(v=>v.lang&&v.lang.startsWith('pl'));
      const u=new SpeechSynthesisUtterance(txt.length<=2?txt+'.':txt);
      u.lang='pl-PL';u.rate=0.9;if(pl)u.voice=pl;
      speechSynthesis.cancel();speechSynthesis.speak(u);
    }
  }
}
</script>

<style>
html,body,#app,.app{height:100%;margin:0}
.app{display:flex;flex-direction:column;align-items:center;justify-content:center;padding:1rem;background:#f7fcff;font-family:'Helvetica Neue',Arial,sans-serif}
.power-btn{position:absolute;top:10px;right:10px;font-size:1.6rem;background:none;border:none;cursor:pointer}
.fade-enter-active,.fade-leave-active{transition:opacity .3s}
.fade-enter-from,.fade-leave-to{opacity:0}
</style>
