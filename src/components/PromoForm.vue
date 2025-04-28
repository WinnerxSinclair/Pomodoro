<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="name">Name</label>
      <input v-focus v-model="form.name" type="text" id="name" minlength="1" maxlength="99" required />
    </div>
    <div>
      <label for="time">Time (in minutes)</label>
      <input v-model="form.time" class="time" id="time" type="number" min="1" max="999" required />
    </div>
    <div>
      <label for="break">Break Time (in minutes)</label>
      <input v-model="form.break" class="time" id="break" type="number" min="1" max="999" required />
    </div>
    <div>
      <label for="reps">Reps</label>
      <input v-model="form.reps" class="time" id="reps" type="number" min="1" max="50" required />
    </div>
    <slot><button>Create</button></slot>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  formVals: { type: Object, required: false }
});


const emit = defineEmits(['submit'])
const form = ref({
  id: props.formVals?.id || crypto.randomUUID(),
  name: props.formVals?.name || '',
  time: props.formVals?.time || 25,
  break: props.formVals?.break || 5,
  reps: props.formVals?.reps || 3,
  completedReps: 0
});

const vFocus = {
  mounted: (el) => el.focus()
}

function submitForm(){
  const payload = { ...form.value };
  emit('submit', payload);
}

</script>

<style scoped>
form{
  width: 100%;
}
form > *{
  display:flex;
  flex-direction: column;
  margin-top: 1rem;
}
input{
  font-size: 2rem;
}
.time{
  max-width: 120px;
}
label{
  font-weight: bold;
}
</style>