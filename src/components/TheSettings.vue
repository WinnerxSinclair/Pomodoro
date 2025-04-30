<template>
  <div class="flex col gap" tabindex="-1" v-focus aria-label="Settings Dialog">
    <section>
      <h3 id="theme-label">Theme</h3>
      <div class="theme-btns" role="radiogroup" aria-labelledby="theme-label">
        <button      
          :class="{ 'selected-theme': themeCheck === theme.value }"
          v-for="theme in themes"
          :key="theme.name"
          @click="changeTheme(theme.value)"
          role="radio"
          :aria-checked="themeCheck === theme.value"
        >
          {{ theme.icon }} {{ theme.name }}
        </button>
      </div>
    </section>
    <section>
      <h3>Controls</h3>
      <div>
        <label for="autoplay_promo_to_break">Autoplay Breaks</label>
        <input
          id="autoplay_promo_to_break"
          type="checkbox"
          v-model="autoPlayBreak"
        >
      </div>
      <div>
        <label for="autoplay_break_to_promo">Autoplay Pomos</label>
        <input
          id="autoplay_break_to_promo"
          type="checkbox"
          v-model="autoPlayPromo"
        >
      </div>
      <div>
        <label for="toggle-pomo-edit-mode">Pomo Edit Mode</label>
        <input
          id="toggle-pomo-edit-mode"
          type="checkbox"
          v-model="pomoEditMode"
        >
      </div>
    </section>
    <section class="default-form">
      <h3>Default Form Values</h3>

      <div class="mt-3">
        <label for="time">Time (in minutes)</label>
        <input
          v-model="form.time"
          class="time"
          id="time"
          type="number"
          min="1"
          max="999"
          required
        />
      </div>

      <div>
        <label for="break">Break Time (in minutes)</label>
        <input
          v-model="form.break"
          class="time"
          id="break"
          type="number"
          min="1"
          max="999"
          required
        />
      </div>

      <div>
        <label for="reps">Reps</label>
        <input
          v-model="form.reps"
          class="time"
          id="reps"
          type="number"
          min="1"
          max="50"
          required
        />
      </div>
    </section>
    <section>

      <h3>Sets</h3>
      <ul
      
        aria-label="select a set"
      >
        <li
          v-for="(set, i) in localSets"
          :key="i"
          class="flex ac sb"
          
          :aria-selected="i === curSet"
        >
          <button
            class="set grow"
            :class="{ 'purple': i === curSet }"
            @click="$emit('select:set', i)"
          >
            {{ set.name }}
          </button>
          <div class="flex">
            <button 
              @click="editSet(set.name, i)"
              aria-label="Edit set {{ set.name }}"
              aria-haspopup="dialog"
              :aria-expanded="showUpdateSetForm.toString()"
              aria-controls="edit-set-form-dialog"
              
            >
              <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000">
                <path
                  d="M0 0h24v24H0V0z"
                  fill="none"
                />
                <path
                  d="M14.06 9.02l.92.92L5.92 19H5v-.92l9.06-9.06M17.66 3c-.25 0-.51.1-.7.29l-1.83 1.83 3.75 3.75 1.83-1.83c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.2-.2-.45-.29-.71-.29zm-3.6 3.19L3 17.25V21h3.75L17.81 9.94l-3.75-3.75z"
                />
              </svg>
            </button>
            <button
              @click="deleteSet(i)"
              aria-label="Delete set {{ set.name }}"
            >
              <svg  xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000">
                <path
                  d="M0 0h24v24H0V0z"
                  fill="none"
                />
                <path
                  d="M16 9v10H8V9h8m-1.5-6h-5l-1 1H5v2h14V4h-3.5l-1-1zM18 7H6v12c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7z" />
              </svg>
            </button>
          </div>
        </li>
      </ul>
      <button
        
        class="set-btn"
        @click="openSetForm"
        aria-haspopup="dialog"
        :aria-expanded="showSetForm.toString()"
        aria-controls="set-form-dialog"
      >
        Add New Set <span aria-hidden="true">+</span>
      </button>
    </section>

    <TheModal
      id="set-form-dialog"
      title="Add Set"
      ariaId="add-set"
      :showModal="showSetForm"
      @close="closeSetForm"
    >
      <SetForm @submit="createSet" />
    </TheModal>

    <TheModal
      id="edit-set-form-dialog"
      title="Edit Set Name"
      ariaId="edit-set-name"
      :showModal="showUpdateSetForm"
      @close="closeEditSetForm"
    >
      <SetForm
        @submit="updateSet"
        :setName="selectedEditSet"
        buttonText="Update"
      />
    </TheModal>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import TheModal from './TheModal.vue';
import SetForm from './SetForm.vue';
const autoPlayBreak = defineModel('autoPlayBreak');
const autoPlayPromo = defineModel('autoPlayPromo');
const pomoEditMode = defineModel('pomoEditMode')

const props = defineProps({
  defaultFormVals: Object,
  sets: Array,
  curSet: Number
});
const emit = defineEmits(['update:df', 'create:set', 'delete:set', 'select:set']);
const themeCheck = ref();
onMounted(() => {
  let temp = localStorage.getItem('theme');
  if (temp) {
    themeCheck.value = temp;
  }
});

const vFocus = {
  mounted: (el) => el.focus()
}
function createSet(set) {
  emit('create:set', set.name);
  showSetForm.value = false;
}

const showSetForm = ref(false);
const showUpdateSetForm = ref(false);
const selectedEditSet = ref(null);
const selectedEditSetIndex = ref(null);


const form = ref({
  time: props.defaultFormVals.time || 25,
  break: props.defaultFormVals.break || 5,
  reps: props.defaultFormVals.reps || 3
});

const localSets = ref(props.sets);


function openSetForm(){
  showSetForm.value = true;
}

function editSet(key, i) {
  selectedEditSet.value = key;
  selectedEditSetIndex.value = i;
  showUpdateSetForm.value = true;
}

function deleteSet(i) {
  emit('delete:set', i);
}

function updateSet(set) {
  localSets.value[selectedEditSetIndex.value].name = set.name;
  showUpdateSetForm.value = false;
}

function closeSetForm(){
  showSetForm.value = false;
}

function closeEditSetForm(){
  showUpdateSetForm.value = false;
}

const themes = [
  { name: 'Light', icon: '🔆', value: 'light' },
  { name: 'Dark', icon: '🌙', value: 'dark' },
  // { name: 'Nature', icon: '🌱', value: 'nature' },
  // { name: 'Cafe', icon: '☕', value: 'cafe' },
]

watch(() => form.value, newVal => emit('update:df', newVal), { deep: true });

function changeTheme(theme) {
  document.documentElement.className = '';
  document.documentElement.classList.add(theme);
  localStorage.setItem('theme', theme);
  themeCheck.value = theme;
}
</script>

<style scoped>
.set-btn {
  margin-top: 1.5rem;
}

h3 {
  text-decoration: underline;
}

.purple {
  color: rgb(206, 63, 241);
}

.set {
  display: block;
  padding: 0;
  background: none;
  box-shadow: none;
  text-align: left;
  font-size: 1.5rem;
  user-select: none;
  flex: 1 1 auto;
  
  overflow-wrap:anywhere;
}

button {
  font-size: 1rem;
}

.default-form>div {
  display: flex;
  flex-direction: column;
  margin-top: 1rem;
}

.selected-theme {
  border: 1px solid var(--text-color);
}

input {
  font-size: 1.5rem;
}

.theme-btns>button:not(:first-child) {
  margin-left: .2rem;
}
</style>