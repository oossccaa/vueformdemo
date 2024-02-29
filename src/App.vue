<script setup>
import { ref,computed } from 'vue'
const step = ref(1)
const data = ref([
  {
    id: 1,
    name: '基本資料',
    fields: [
      { key: 'Name', value: '', type: 'input' },
      { key: 'Gender', value: '', type: 'radio', options: ['Male', 'Female', 'Other'] },
      { key: 'Languages', value: [], type: 'checkbox', options: ['JavaScript', 'Python', 'Java', 'C++'] },
    ],
    next: 2
  },
  {
    id: 2,
    name: '聯絡方式',
    fields: [
      { key: 'Email', value: '', type: 'input' },
      { key: 'Phone', value: '', type: 'input' },
      { key: 'Address', value: '', type: 'textarea' }
    ],
    next: 3
  },
  {
    id:3,
    name: '意見回饋',
    fields: [
      { key: 'Feedback', value: '', type: 'textarea' }
    ],
    next: 0
  }
])

const currentStep = computed(() => {
  return data.value[step.value - 1]
})

const saveData = ref([]);

const submitForm = () => {

      if (currentStep.value.next !== 0) {
        step.value = currentStep.value.next;
      } else {
        step.value = 0;
        saveData.value = data.value.map(item => {
          return item.fields.reduce((prev, curr) => {
            return {
           ...prev,
              [curr.key]: curr.value
            }
          },{})
        })
        console.log('All Form Data:', saveData);
      }
    };

</script>

<template>
  <div v-if="step!=0" >
    <div v-for="(field, index) in currentStep.fields" :key="index">
      <label>{{ field.key }}</label>
      <template v-if="field.type === 'radio'">
        <div v-for="(option, optionIndex) in field.options" :key="optionIndex">
          <input type="radio" :value="option" v-model="field.value">
          <label>{{ option }}</label>
        </div>
      </template>
      <template v-else-if="field.type === 'checkbox'">
        <div v-for="(option, optionIndex) in field.options" :key="optionIndex">
          <input type="checkbox" :value="option" v-model="field.value">
          <label>{{ option }}</label>
        </div>
      </template>
      <template v-else-if="field.type === 'input'">
        <input type="text" v-model="field.value">
      </template>
      <template v-else-if="field.type === 'textarea'">
        <textarea v-model="field.value"></textarea>
      </template>
    </div>
    <button type="submit" @click="submitForm">{{ currentStep.next !== 0 ? 'Next' : 'Submit' }}</button>
  </div>
  <div v-else>
    <div class="save-data" v-if="saveData.length > 0">
      <div class="step-data" v-for="(stepData, index) in saveData" :key="index">
        <h3>{{ stepData.stepName }}</h3>
        <ul>
          <li v-for="(value, key) in stepData" :key="key">{{ key }}: {{ value }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
  body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    margin: 0;
    padding: 0;
  }
  #app {
    max-width: 600px;
    margin: 20px auto;
    background-color: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  }
  form {
    display: grid;
    gap: 10px;
  }
  label {
    font-weight: bold;
    text-align: left;
  }
  input[type="text"],
  textarea {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
  }
  input[type="radio"],
  input[type="checkbox"] {
    margin-right: 5px;
  }
  button[type="submit"] {
    padding: 10px 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  button[type="submit"]:hover {
    background-color: #0056b3;
  }
</style>
