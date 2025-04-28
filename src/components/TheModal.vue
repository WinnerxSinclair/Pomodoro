<template>
  <Transition>
    <div class="outer" v-if="showModal">
      <div class="inner" @click.stop @mousedown.stop>
        <slot />
      </div>
      <div class="x"></div>
    </div>
  </Transition>
</template>

<script setup>
const props = defineProps({
  showModal: {
    type: Boolean,
    default: false
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
  width: clamp(350px, 50%, 450px);
  background:var(--accent-bg);
  border-radius: 1rem;
  padding: 1vw 5vh;
  padding-bottom: 3rem;
  display:flex;
  justify-content: center;
  align-items: center;
}
.x{
  position: absolute;
  top: 5px;
  right: 5px;
  width: 60px;
  aspect-ratio: 1;
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