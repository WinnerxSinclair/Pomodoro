<template>
  <Transition @after-leave="handleLeave">
    <div ref="dialog" tabindex="-1" class="outer" v-if="showModal" @mousedown.self="$emit('close')" 
      @keydown.esc.stop.prevent="$emit('close')" role="dialog" aria-modal="true" :aria-labelledby="ariaId"
    >
      <h2 :id="ariaId" class="hidden">{{ title }}</h2>
      <div class="inner" >
        <slot />
      </div>
      <button @click.stop="$emit('close')" class="x" aria-label="Close Modal"></button>
    </div>
  </Transition>
</template>

<script setup>
import { ref, watch, useTemplateRef, watchEffect } from 'vue'
const props = defineProps({
  showModal: {
    type: Boolean,  
    default: false
  },
  title: String,
  ariaId: String
});


const dialog = useTemplateRef('dialog');
const emit = defineEmits(['close'])

const opener = ref(null);
watch(() => props.showModal, (newVal) => {
  if(newVal) opener.value = document.activeElement;
  else {
    if (trap && dialog.value) {
      dialog.value.removeEventListener('keydown', trap)
    }
    trap = null
  }

})

function handleLeave(){
  opener.value?.focus();
}

let trap = null;

watchEffect(() => {
  if(dialog.value){
    // dialog.value.focus();
    trap = (e) => {
      if (e.key !== 'Tab') return 
      const focusables = dialog.value.querySelectorAll('button, [href], input,' +
        'select, textarea, [tabindex]:not([tabindex="-1"])')
      const firstEl = focusables[0]
      const lastEl  = focusables[focusables.length - 1]

      if (e.shiftKey && e.target === firstEl) {        // Shift-Tab on first
        e.preventDefault()
        lastEl.focus()
      } else if (!e.shiftKey && e.target === lastEl) { // Tab on last
        e.preventDefault()
        firstEl.focus()
      }
    }
    dialog.value.addEventListener('keydown', trap);
   
  }
})


</script>

<style scoped>
.outer{
  position:absolute;
  inset: 0;
  display: flex;
  justify-content: center;
  z-index: 99;
  backdrop-filter: blur(10px);
  background: rgba(0, 0, 0, 0.26);
}
.inner{
  margin-top: 5rem;
  height: fit-content;
  width: clamp(350px, 40%, 450px);
  background:var(--accent-bg);
  border-radius: 1rem;
  padding: 1rem 3rem 2rem 3rem;
  display:flex;
  justify-content: center;
  align-items: center;
}
.inner > :first-child{
  width: 100%;
}
.x{
  position: absolute;
  top: 5px;
  right: 5px;
  width: 60px;
  aspect-ratio: 1;
  padding: 0;
  background: transparent;
}
.x::before, .x::after{
  content:'';
  background:white;
  position:absolute;
  left: 50%;
  top: 50%;
  width: 5px;
  height: 100%;
}
.x::before{
  transform: translate(-50%, -50%) rotate(45deg);
}
.x::after{
  transform: translate(-50%, -50%) rotate(-45deg);
}
.v-enter-active, .v-leave-active{
  transition: all .3s ease-in-out;
}

.v-enter-from, .v-leave-to{
  opacity: 0;
}


.v-enter-active .inner, .v-leave-active .inner{
  transition: all .3s ease-in-out;
}
.v-enter-from .inner, .v-leave-to .inner{
  opacity: 0;
  transform: translateX(-200px);
}

</style>