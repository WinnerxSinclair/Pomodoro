<template>
  <button @click="showSettings = true" aria-label="settings">
    <svg class="settings" xmlns="http://www.w3.org/2000/svg" height="32px" viewBox="0 0 24 24" width="32px" fill="#000000"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M19.43 12.98c.04-.32.07-.64.07-.98 0-.34-.03-.66-.07-.98l2.11-1.65c.19-.15.24-.42.12-.64l-2-3.46c-.09-.16-.26-.25-.44-.25-.06 0-.12.01-.17.03l-2.49 1c-.52-.4-1.08-.73-1.69-.98l-.38-2.65C14.46 2.18 14.25 2 14 2h-4c-.25 0-.46.18-.49.42l-.38 2.65c-.61.25-1.17.59-1.69.98l-2.49-1c-.06-.02-.12-.03-.18-.03-.17 0-.34.09-.43.25l-2 3.46c-.13.22-.07.49.12.64l2.11 1.65c-.04.32-.07.65-.07.98 0 .33.03.66.07.98l-2.11 1.65c-.19.15-.24.42-.12.64l2 3.46c.09.16.26.25.44.25.06 0 .12-.01.17-.03l2.49-1c.52.4 1.08.73 1.69.98l.38 2.65c.03.24.24.42.49.42h4c.25 0 .46-.18.49-.42l.38-2.65c.61-.25 1.17-.59 1.69-.98l2.49 1c.06.02.12.03.18.03.17 0 .34-.09.43-.25l2-3.46c.12-.22.07-.49-.12-.64l-2.11-1.65zm-1.98-1.71c.04.31.05.52.05.73 0 .21-.02.43-.05.73l-.14 1.13.89.7 1.08.84-.7 1.21-1.27-.51-1.04-.42-.9.68c-.43.32-.84.56-1.25.73l-1.06.43-.16 1.13-.2 1.35h-1.4l-.19-1.35-.16-1.13-1.06-.43c-.43-.18-.83-.41-1.23-.71l-.91-.7-1.06.43-1.27.51-.7-1.21 1.08-.84.89-.7-.14-1.13c-.03-.31-.05-.54-.05-.74s.02-.43.05-.73l.14-1.13-.89-.7-1.08-.84.7-1.21 1.27.51 1.04.42.9-.68c.43-.32.84-.56 1.25-.73l1.06-.43.16-1.13.2-1.35h1.39l.19 1.35.16 1.13 1.06.43c.43.18.83.41 1.23.71l.91.7 1.06-.43 1.27-.51.7 1.21-1.07.85-.89.7.14 1.13zM12 8c-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4-1.79-4-4-4zm0 6c-1.1 0-2-.9-2-2s.9-2 2-2 2 .9 2 2-.9 2-2 2z"/></svg>
  </button>
  
  <TheModal :showModal="showPromoForm" @close="showPromoForm = false" title="Create Pomo" ariaId="create-pomo">
    <PromoForm @submit="createPromo" :formVals="defaultForm" />
  </TheModal>
  <TheModal :showModal="showEditPromoForm" @close="showEditPromoForm = false" title="Edit Pomo" ariaId="edit-pomo">
    <PromoForm @submit="updatePromo" :formVals="curPromo" btnString="Update"></PromoForm>
  </TheModal>

  <TheModal :showModal="showSettings" @close="showSettings = false" >
    <TheSettings 
      v-model:auto-play-break="controls.autoPlayBreak" 
      v-model:auto-play-promo="controls.autoPlayPromo"
      v-model:pomo-edit-mode="controls.pomoEditMode"
      :defaultFormVals="defaultForm"
      :sets="sets"
      :curSet="currentSet"
      title="Settings"
      ariaId="settings"
      @update:df="updateDefaultForm"
      @create:set="createSet"
      @delete:set="deleteSet"
      @select:set="changeSet"
     
    />
  </TheModal>

  <main>
    <h1 class="hidden">Pomodoro Timer App</h1>
    <div 
      class="tac fs-300" 
      v-if="curPromo && curPromo.name"
    >
      {{ curPromo.name }} {{ runtime.phase === 'break' ? '- break' : ''}}
    </div>
    <div class="flex ac jc">
      <div class="timer" :class="background" role="timer" aria-live="polite">{{ mm }}:{{ ss }}</div>
    </div>
    <div class="tac">
      
      <div v-if="curPromo">
        <button @click="reset(runtime.index)" aria-label="Reset Timer"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></button>
        <button class="play-btn" v-if="runtime.status !== 'running'" @click="start(runtime.index)" aria-label="Start/Resume Timer"><svg class="big-svg" xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><path d="M12,2C6.48,2,2,6.48,2,12s4.48,10,10,10s10-4.48,10-10S17.52,2,12,2z M12,20c-4.41,0-8-3.59-8-8s3.59-8,8-8s8,3.59,8,8 S16.41,20,12,20z M9.5,16.5l7-4.5l-7-4.5V16.5z"/></g></svg></button>
        <button class="play-btn" v-else @click="pause" aria-label="Pause Timer"><svg class="big-svg" xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><g><path d="M9,16h2V8H9V16z M12,2C6.48,2,2,6.48,2,12s4.48,10,10,10s10-4.48,10-10S17.52,2,12,2z M12,20c-4.41,0-8-3.59-8-8 s3.59-8,8-8s8,3.59,8,8S16.41,20,12,20z M13,16h2V8h-2V16z"/></g></g></svg></button>
        <button @click="skip(runtime.index)" aria-label="Skip Timer"><svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><g><polygon points="6.41,6 5,7.41 9.58,12 5,16.59 6.41,18 12.41,12"/><polygon points="13,6 11.59,7.41 16.17,12 11.59,16.59 13,18 19,12"/></g></g></svg></button>
      </div>
      <div v-else>No Pomos Yet, Add One Below!</div>
      
      <div v-if="setsLength">
        <button @click="showPromoForm = true" class="mt-3">Add Pomo</button>
        <button @click="resetReps" v-if="sets[currentSet].promos.length > 0">Reset Reps</button>
      </div>
      <div v-else>
        <div>You have no Sets! Pomos must go in sets. Add one below.</div>
        
        <button class="block" @click="showSettings = true">Settings</button>
      </div>
    </div>
    <ul v-if="sets.length > 0" aria-label="select a pomo">
      
      <li class="flex gap" v-for="(promo,i) in sets[currentSet].promos" :key="promo.id">
        <button class="promo" :class="[i === runtime.index ? `selected-promo ${background}` : '']" @click="selectPromo(i, $event)">
          <div class="promo-name">{{ promo.name }}</div>
          <div class="flex ac sub-btn-controls">
            <button v-if="runtime.status !== 'running' || runtime.index !== i" @click="start(i)" aria-label="Start/Resume Timer"><svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><path d="M12,2C6.48,2,2,6.48,2,12s4.48,10,10,10s10-4.48,10-10S17.52,2,12,2z M12,20c-4.41,0-8-3.59-8-8s3.59-8,8-8s8,3.59,8,8 S16.41,20,12,20z M9.5,16.5l7-4.5l-7-4.5V16.5z"/></g></svg></button>
            <button v-else @click="pause" aria-label="Pause Timer"><svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><g><path d="M9,16h2V8H9V16z M12,2C6.48,2,2,6.48,2,12s4.48,10,10,10s10-4.48,10-10S17.52,2,12,2z M12,20c-4.41,0-8-3.59-8-8 s3.59-8,8-8s8,3.59,8,8S16.41,20,12,20z M13,16h2V8h-2V16z"/></g></g></svg></button>
            <button v-if="runtime.index === i" @click="reset(i)" aria-label="Reset Timer"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"/></svg></button>
            <button v-if="runtime.index === i" @click="skip(i)" aria-label="Skip Timer"><svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><g><rect fill="none" height="24" width="24"/></g><g><g><polygon points="6.41,6 5,7.41 9.58,12 5,16.59 6.41,18 12.41,12"/><polygon points="13,6 11.59,7.41 16.17,12 11.59,16.59 13,18 19,12"/></g></g></svg></button>
          </div>
          <div class="reps">{{ promo.completedReps }} / {{ promo.reps }}</div>
        </button>
        <div class="flex ac gap-0" v-if="controls.pomoEditMode">
          <button @click="editPromo(i)" aria-label="Edit Pomo">
            <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M14.06 9.02l.92.92L5.92 19H5v-.92l9.06-9.06M17.66 3c-.25 0-.51.1-.7.29l-1.83 1.83 3.75 3.75 1.83-1.83c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.2-.2-.45-.29-.71-.29zm-3.6 3.19L3 17.25V21h3.75L17.81 9.94l-3.75-3.75z"/></svg>
          </button>
          <button @click="deletePromo(i)" aria-label="Delete Pomo">
            <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><path d="M0 0h24v24H0V0z" fill="none"/><path d="M16 9v10H8V9h8m-1.5-6h-5l-1 1H5v2h14V4h-3.5l-1-1zM18 7H6v12c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7z"/></svg>
          </button>
        </div>
      </li>
    </ul>
  </main>
</template>

<script setup>
import { ref, onMounted, computed, watch, reactive } from 'vue'
import PromoForm from './components/PromoForm.vue'
import TheModal from './components/TheModal.vue'
import TheSettings from './components/TheSettings.vue';

function playSound(){
  const sound = new Audio('/alert.mp3');
  sound.play().catch(err => {
    console.warn('Couldn’t play sound:', err);
  })
}
const showPromoForm = ref(false);
const showEditPromoForm = ref(false);
const showSettings = ref(false);
const sets = ref([{ name: 'set1', promos: [] }]);
const currentSet = ref(0);

const defaultForm = ref({
  time: 25,
  break: 5,
  reps: 3
});

const runtime = reactive({
  status: 'idle',
  phase: 'promo',
  index: -1,
  leftMs: 0,
  id: null
});

//computeds
const mm = computed(() => (Math.floor(runtime.leftMs/60000)).toString());
const ss = computed(() => (Math.floor(runtime.leftMs/1000)%60).toString().padStart(2,'0'));
const curPromo = computed(() => {
  if(runtime.index === -1 || !setsLength.value) return;
  return sets.value[currentSet.value].promos[runtime.index] || {};
});

const setsLength = computed(() => sets.value.length > 0)


function resetReps(){
  sets.value[currentSet.value].promos.map((p) => {
    p.completedReps = 0;
    return p;
  })
}

function updateDefaultForm(payload){
  defaultForm.value = payload;
  localStorage.setItem('defaultForm', JSON.stringify(payload));
}

function runTimer(duration, onDone){
  clearInterval(runtime.id);
  const start = Date.now();
  runtime.leftMs = duration;
  runtime.status = 'running';

  runtime.id = setInterval(() => {
    runtime.leftMs = duration - (Date.now() - start);
    if(runtime.leftMs <= 0){
      clearInterval(runtime.id);
      fireNotification();
      onDone?.();
    }
  }, 1000)
}

function fireNotification(){
  if(Notification.permission !== 'granted') return;
  let message = runtime.phase === 'promo' ? 'Pomo Completed!' : 'Breaktime over!';
  const options = {
    body: message,
    vibrate: [200, 100, 200] 
  };

  const notif = new Notification('Timer Complete!', options);

  playSound();
}

function stopTimer(duration){
  clearInterval(runtime.id);
  runtime.leftMs = duration;
  runtime.status = 'idle';
}


function skip(idx){
  const p = sets.value[currentSet.value].promos[idx];
  clearInterval(runtime.id);
  if(runtime.phase === 'promo'){
    p.completedReps++;
    runtime.phase = 'break';
    runtime.leftMs = p.break*60000;
  }else{
    runtime.phase = 'promo';
    runtime.leftMs = p.time*60000;
  }
  runtime.status = 'idle';
}

function editPromo(idx){
  clearInterval(runtime.id);
  const p = sets.value[currentSet.value].promos[idx];
  if(!p) return;
  Object.assign(runtime, { status: 'idle', phase: 'promo', index: idx, leftMs: p.time*60000 });
  showEditPromoForm.value = true;
}

function createPromo(promo){
  sets.value[currentSet.value].promos.push(promo);
  showPromoForm.value = false;
  const index = sets.value[currentSet.value].promos.length-1;
  if(sets.value[currentSet.value].promos.length === 1) selectPromo(index);
  
}

function updatePromo(promo){
  sets.value[currentSet.value].promos = sets.value[currentSet.value].promos.map((p) => {
    if(p.id === promo.id) p = promo;
    return p;
  });
  runtime.leftMs = promo.time*60000;
  showEditPromoForm.value = false;
}

function deletePromo(idx){
  sets.value[currentSet.value].promos.splice(idx, 1);
  if (idx === runtime.index) {
    let len = sets.value[currentSet.value].promos.length;
    if(!len) resetRuntime();
    else selectPromo(0);
  }else if(idx < runtime.index) runtime.index--;
  

}

function resetRuntime(){
  clearInterval(runtime.id);
  Object.assign(runtime, { status: 'idle', phase: 'promo', index: -1, leftMs: 0 });
}

function createSet(set){
  sets.value.push({ name: set, promos: [] });
}

function deleteSet(i){
  if(currentSet.value === i){
    currentSet.value = 0;
    let len = sets.value[currentSet.value].promos.length;
    if(!len) resetRuntime();
    else selectPromo(0);
  }else if(currentSet.value > i) currentSet.value--;
  sets.value.splice(i, 1);
}


function changeSet(i){
  if(currentSet.value === i) return;
  currentSet.value = i;
  let idx = sets.value[i].promos.length > 0 ? 0 : -1;
  
  selectPromo(idx);
}


function selectPromo(idx, evt){
  const fromClick = evt instanceof MouseEvent;

  if(fromClick && idx === runtime.index) return;
  clearInterval(runtime.id);
  const p = sets.value[currentSet.value].promos[idx];
  
  if(!p){
    Object.assign(runtime, { status: 'idle', phase: 'promo', index: idx, leftMs: 0 })
  }else{
    Object.assign(runtime, { status: 'idle', phase: 'promo', index: idx, leftMs: p.time*60000 });
  }
  
}

const controls = ref({
  autoPlayBreak: false,
  autoPlayPromo: false,
  pomoEditMode: false,
});

watch(() => controls.value, newVal => localStorage.setItem('controls', JSON.stringify(newVal)), { deep: true });


function start(idx, auto_break_flag = true, auto_promo_flag = true){
  idx = idx ?? runtime.index;
  const p = sets.value[currentSet.value].promos[idx];
  if(!p) return;

  if(Notification.permission === 'default'){
    Notification.requestPermission().then(permission => {
      console.log('Notification persmission:', permission);
    });
  }else{
    console.log('already notifcaiotn permission', Notification.permission);
  }
  let dur = (runtime.status === 'paused' && runtime.index === idx)
            ? runtime.leftMs
            : (runtime.phase === 'promo' ? p.time : p.break)*60000;
  
  if(runtime.index !== idx){
    dur = p.time*60000;
    runtime.phase = 'promo';
  }

  if(!auto_break_flag && runtime.phase === 'break'){
    stopTimer(dur);
    return;
  };
  if(!auto_promo_flag && runtime.phase === 'promo'){
    stopTimer(dur);
    return;
  };
  runTimer(dur, () => {
    if(runtime.phase === 'promo') p.completedReps++;
    runtime.phase = runtime.phase === 'promo' ? 'break' : 'promo';
    start(idx, controls.value.autoPlayBreak, controls.value.autoPlayPromo);
  });

  runtime.index = idx;
}

function pause(){
  runtime.status = 'paused';
  clearInterval(runtime.id);
}

function reset(idx){
  clearInterval(runtime.id);
  const p = sets.value[currentSet.value].promos[idx];
  if(!p) return;
  runtime.status = 'idle';
  runtime.leftMs = (runtime.phase === 'promo' ? p.time : p.break)*60000;
}

const background = computed(() => {
  return runtime.status === 'running' ? 'running' : 'paused';
});

watch(
  () => sets.value,
  v => localStorage.setItem('sets', JSON.stringify(v)),
  { deep: true }
)

watch(currentSet, v => localStorage.setItem('lastSet', JSON.stringify(v)))

onMounted(() => {
  let localSets = JSON.parse(localStorage.getItem('sets'));
  if(localSets) sets.value = localSets;
  if(sets.value.length === 0) sets.value = [{ name: 'set1', promos: [] }];
  let localCurSet = JSON.parse(localStorage.getItem('lastSet'));
  if(localCurSet) currentSet.value = localCurSet;
  console.log(currentSet.value)
  if(sets.value[currentSet.value].promos.length > 0) selectPromo(0);

  let ap = JSON.parse(localStorage.getItem('controls'));
  if(ap) controls.value = ap;

  let df = JSON.parse(localStorage.getItem('defaultForm'));
  if(df) defaultForm.value = df;
});

</script>

<style scoped>
.promo{
  font-size: 2rem;
  display:grid;
  width: fit-content;
  grid-template-columns: 400px 200px auto;
  align-items: center;
  border-bottom: 2px solid transparent;
  background:none;
  box-shadow: none;
  padding: 0;
  text-align: left;
  padding: .2rem 1rem;
  border-radius: 1rem;;
  cursor: default;
}

li{
  margin-top: 1.5rem;
}

.promo:hover{
  border-bottom: 2px solid var(--text-color);
  background: var(--accent-bg);
  transition:none;
}

.timer{
  font-size: 6rem;
  font-family: "Share Tech Mono";
  padding-left: 2rem;
  padding-right: 2rem; 
  background: var(--accent-bg);
  border: 1px dashed;
  border-radius: 9999px;
}



.addpromo{
  text-decoration: underline;
  cursor: pointer;
  transition: none;
}

.addpromo:hover{
  color: rgb(255, 184, 255);
}

button > svg{
  vertical-align: middle;
}

.sub-btn-controls > button{
  background: transparent;
  border:none;
  box-shadow:none;
}

button:hover{
  background: rgba(170, 170, 170, 0.445);
}

.sub-btn-controls svg{
  width: 30px;
  height: 30px;
}

.big-svg{
  width: 60px;
  height: 60px;
}

ul{
  margin-top: 4rem;
}

.running{
  border-color:  rgb(255, 177, 177);
}

.paused{
  border-color:  rgb(160, 209, 255);
}

.selected-promo{
  background: var(--accent-bg);
  border-bottom-width: 2px;
  border-bottom-style: solid;
}

.play-btn{
  margin-left: 1rem;
  margin-right: 1rem;
}

@media(max-width: 725px){
  .sub-btn-controls{
    display:none;
  }
  .promo{
    display:flex;
    width: 100%;
    font-size: 1.2rem;
    justify-content: space-between;
    padding: .5rem 1rem;
  }
  .promo-name{
    flex: 1 1 auto;
    overflow-wrap:anywhere;
  }
  .reps{
    flex: 0 0 auto;
    margin-left: 1rem;
  }
}
</style>
