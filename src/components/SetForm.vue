<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="name">Set Name</label>
      <input 
        v-focus 
        v-model.trim="form.name" 
        type="text" id="name" 
        name="name" minlength="1" 
        maxlength="99" 
        required
        aria-required="true" 
      />
    </div>
    <button>{{ buttonText || 'Create' }}</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['submit']);

const props = defineProps({
  setName: String,
  buttonText: String
})
const form = ref({
  name: props.setName || '',
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
button{
  font-size: 1rem;
  margin-top: 2rem;
}
</style>