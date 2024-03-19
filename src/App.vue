<script setup>
import { ref, computed } from 'vue'
import Customeized from './Customized.vue'
const step = ref(1)
// option 可以是string也可以是object, string的話就會直接顯示label以及value, object的話會包含value以及contents陣列
// visible 可以是string也可以是object, string的話就表示對應key欄位的value有值就會顯示, object的話就表示value必須符合key的value才會有值
// 'key' or {visible: {key: 'key', value: 'value'}}
const data = ref([
  {
    id: 1,
    name: 'Form',
    fields: [
      { key: 'conditions', label: 'Underlying medical/psychiatric conditions.', value: '', type: 'text' },
      { key: 'medication', label: 'What medication(s) are requesting to be filled?', 
        value: '', 
        type: 'select', 
        options: [
          'Aspirin',
          'Cocaine',
          'Oxycodone',
        ],
        inputs: [
          {
            key: 'MedicationRoute',prefix: 'Medication (Route)',value: '',type: 'text',
          },
          {
            key: 'Dosage',prefix: 'Dosage',value: '',type: 'text',
          },
          {
            key: 'Direction',prefix: 'Direction',value: '',type: 'text',
          },
          {
            key: 'Quantity', prefix: 'Quantity',value: '',type: 'text',
          }
        ]
      },
      {
        key: 'For medication', 
        label: 'For medication %medication. Are you regularly taking this medication?',
        value: '', 
        type: 'radio', 
        options: ['Yes, I take it regularly.', 'No, I take it occasionally.','No, I take it as needed.',{
          value: 'customized',
          contents: [
            {
              key: 'discontinued', prefix: ' No, I have discontinued for',value: '',type: 'text',suffix: ' months.',
            }
          ]
        }],
        visible:'medication',
      },
      {
        key: 'symptoms',
        label: 'Do you have any of the following symptoms?',
        value: {},
        type: 'true-false',
        options: ['Burning urination (Dysuria)','Frequent urination','Intense urge to urinate','Intense urge to urinate','Flank (back) pain',
          {
            value: 'customized',
            contents: [
              {
                key: 'color',prefix: 'Genital discharge. Color:',value: '',type: 'text',suffix: ', ',
              },
              {
                key: 'texture',prefix: 'Texture:',value: '',type: 'text',suffix: '.'
              },
              {
                key: 'othersymptoms',prefix: 'Other symptoms:',value: '',type: 'text',suffix: '.'
              }
            ]
          }
        ]
      },
      {
        key: 'lastDiagnostic',label: 'Last diagnostic test:', value: '', type: 'radio',
        options: [
          {
            value: 'customized',
            contents: [
              {
                key: 'test',
                prefix: 'Type of Test:',
                value: '',
                type: 'text',
                suffix: '. ',
              },
              {
                key: 'date',
                prefix: 'Date:',
                value: '',
                type: 'text'
              },
            ]
          },
          'Not applicable.'
        ]
      },
    ],
    next: 0
  },
  // {
  //   id: 2,
  //   name: '聯絡方式',
  //   fields: [
  //     { key: 'Email', value: '', type: 'text' },
  //     { key: 'Phone', value: '', type: 'text' },
  //     { key: 'Address', value: '', type: 'textarea' }
  //   ],
  //   next: 3
  // },
  // {
  //   id: 3,
  //   name: '意見回饋',
  //   fields: [
  //     { key: 'Feedback', value: '', type: 'textarea' }
  //   ],
  //   next: 0
  // },
])

const currentStep = computed(() => {
  return data.value[step.value - 1]
})

const isCustomized = (option)=>{
  return typeof option !== 'string'
}

const __formatLabel = (label,fields)=>{
  console.log(label);
  return label.replace(/%(\w+)/g, (match, key) => {
    const field = fields.find(item => item.key === key);
    if (field) {
      return field.value;
    } else {
      return match;
    }
  });
}

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
      }, {})
    })
    console.log('All Form Data:', saveData);
  }
};
const showFields = (field, fields) => {
  if (!field.visible) {
    return true; // 沒有設置 visible 屬性，默認顯示
  }
  if(typeof field.visible ==='string') {
    return fields.find(f=>f.key===field.visible).value
  }
  const { key, value } = field.visible;
  return fields.some(f => f.key === key && f.value === value);
};
</script>

<template>
  <div v-if="step != 0">
    <div v-for="(field, index) in currentStep.fields" :key="index" v-show="showFields(field, currentStep.fields)">
      <label>{{ __formatLabel(field.label,currentStep.fields) }}</label>
      <template v-if="field.type === 'radio'">
        <div v-for="(option, optionIndex) in field.options" :key="optionIndex">
          <label>
            <input type="radio" :value="isCustomized(option) ? option.value : option" v-model="field.value" />
            <template v-if="isCustomized(option)">
              <Customeized :option="option" />
            </template>
            <span v-else>{{ option }}</span>
          </label>
        </div>
      </template>
      <template v-else-if="field.type === 'checkbox'">
        <div v-for="(option, optionIndex) in field.options" :key="optionIndex">
          <input type="checkbox" :value="option" v-model="field.value">
          <label>{{ option }}</label>
        </div>
      </template>
      <template v-else-if="field.type === 'text'">
        <input type="text" v-model="field.value">
      </template>
      <template v-else-if="field.type === 'textarea'">
        <textarea v-model="field.value"></textarea>
      </template>
      <template v-else-if="field.type === 'true-false'">
        <div v-for="(option, index) in field.options" :key="index">
          <label>
            <input type="radio" :value="true" v-model="field.value[index]">
            Y
          </label>
          <label>
            <input type="radio" :value="false" v-model="field.value[index]">
            N
          </label>
          <template v-if="isCustomized(option)">
              <Customeized :option="option" />
            </template>
          <span v-else>{{ option }}</span>
        </div>
      </template>
      <template v-else-if="field.type ==='select'">
        <select v-model="field.value">
          <option v-for="(option, optionIndex) in field.options" :key="optionIndex">{{ option }}</option>
        </select>
      </template>
      <div v-if="field.inputs">
        <div v-for="(input, inputIndex) in field.inputs" :key="inputIndex">
          <label>{{ input.prefix }}</label>
          <input type="text" class="blank" v-model="input.value">
          <label>{{ input.suffix }}</label>
        </div>
      </div>
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

<style lang="scss" scoped>
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
  box-sizing: border-box;
  &.blank {
  display: inline;
  width: 100px;
  outline: none;
  border: unset;
  border-bottom: 1px solid #ccc;
  margin: 0 5px
}
}

select{
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
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
