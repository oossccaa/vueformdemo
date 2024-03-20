<script setup>
import { ref, computed,nextTick } from 'vue'
import Multiselect from 'vue-multiselect'
import Customeized from './Customized.vue'
const step = ref(1)
const selectRef = ref(null);
// option 可以是string也可以是object, string的話就會直接顯示label以及value, object的話會包含value以及contents陣列
// visible 可以是string也可以是object, string的話就表示對應key欄位的value有值就會顯示, object的話就表示value必須符合key的value才會有值
// 'key' or {visible: {key: 'key', value: 'value'}}
const data = ref([
  {
    id: 1,
    name: 'Form',
    fields: [
      { key: 'conditions', label: 'Underlying medical/psychiatric conditions.', value: '', type: 'text' },
      {
        key: 'medication', label: 'What medication(s) are requesting to be filled?',
        value: '',
        type: 'medication',
        options: [
          'Aspirin',
          'Cocaine',
          'Oxycodone',
        ],
        list: [],
        extend:  {
          key: 'For medication',
          label: 'For medication %medication. Are you regularly taking this medication?',
          value: '',
          type: 'radio',
          options: ['Yes, I take it regularly.', 'No, I take it occasionally.', 'No, I take it as needed.', {
            value: 'customized',
            contents: [
              {
                key: 'discontinued', prefix: 'No, I have discontinued for', value: '', type: 'text', suffix: ' months.',
              }
            ]
          }],
        },
      },
      {
        key: 'symptoms',
        label: 'Do you have any of the following symptoms?',
        value: {},
        type: 'true-false',
        options: ['Burning urination (Dysuria)', 'Frequent urination', 'Intense urge to urinate', 'Intense urge to urinate', 'Flank (back) pain',
          {
            value: 'customized',
            contents: [
              {
                key: 'color', prefix: 'Genital discharge. Color:', value: '', type: 'text', suffix: ', ',
              },
              {
                key: 'texture', prefix: 'Texture:', value: '', type: 'text', suffix: '.'
              },
              {
                key: 'othersymptoms', prefix: 'Other symptoms:', value: '', type: 'text', suffix: '.'
              }
            ]
          }
        ]
      },
      {
        key: 'lastDiagnostic', label: 'Last diagnostic test:', value: '', type: 'radio',
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

const isCustomized = (option) => {
  return typeof option !== 'string'
}

const currentStep = computed(() => {
  return data.value[step.value - 1]
})

const __formatLabel = (label, name) => {
  return label.replace(/%medication/g, (match, key) => {
    if (name) {
      return name;
    } else {
      return match;
    }
  });
}

const saveData = ref([]);

const handleCreateMedication = async (value,index,extend)=>{
  if(!value || currentStep.value.fields[index].list.some(item=>item.name === value)){
    return
  }
  currentStep.value.fields[index].list.push({
    name: value,
    dosage: '',
    direction: '',
    quantity: '',
    extend: JSON.parse(JSON.stringify(extend))
  })
  selectRef.value[0].selectedIndex = 0;
  await nextTick();
}

const handleSelectExtend = (m,e)=>{
  console.log(m,e.target.value);
}

const submitForm = () => {

  if (currentStep.value.next !== 0) {
    step.value = currentStep.value.next;
  } else {
    step.value = 0;
    saveData.value = data.value.map(item => {
      return item.fields.reduce((prev, { type, value, options }) => {
        if (['radio', 'checkbox'].includes(type)) {
          if (value) {

          }
        }
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
  if (typeof field.visible === 'string') {
    return fields.find(f => f.key === field.visible).value
  }
  const { key, value } = field.visible;
  return fields.some(f => f.key === key && f.value === value);
};
</script>

<template>
  <div class="flex">
    <div class="step-list">
      Step: {{ step }}
    </div>
    <div class="step-form">
      <div v-if="step != 0">
        <div class="step-form__item" v-for="(field, index) in currentStep.fields" :key="index" v-show="showFields(field, currentStep.fields)">
          <label>{{field.label}}</label>
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
          <template v-else-if="field.type === 'medication'">
            <select ref="selectRef" @change="e=>handleCreateMedication(e.target.value,index,field.extend)">
              <option value="null"></option>
              <option v-for="(option, optionIndex) in field.options" :key="optionIndex">{{ option }}</option>
            </select>
            <template v-if="field.list.length">
              <div class="medication__item" v-for="(medication, inputIndex) in field.list" :key="inputIndex">
                <div>
                  <label>Medication (Route)</label>
                  <input type="text" class="blank bold" v-model="medication.name" />
                  <label>Dosage</label>
                  <input type="text" class="blank" v-model="medication.dosage" />
                  <label>Direction</label>
                  <input type="text" class="blank" v-model="medication.direction" />
                  <label>Quantity</label>
                  <input type="text" class="blank" v-model="medication.quantity" />
                </div>
                <div class="bold my-4">{{ __formatLabel(medication.extend.label, medication.name)  }}</div>
                <div v-for="(option, optionIndex) in medication.extend.options" :key="optionIndex">
                  <label>
                    <input type="radio" :value="optionIndex" v-model="medication.extend.value" />
                    <template v-if="isCustomized(option)">
                      <Customeized :option="option" />
                    </template>
                    <span v-else>{{ option }}</span>
                  </label>
                </div>
              </div>
            </template>

          </template>
        </div>
        <!-- <button type="submit" @click="submitForm">{{ currentStep.next !== 0 ? 'Next' : 'Submit' }}</button> -->
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
    </div>
  </div>
</template>

<style lang="scss" scoped>
*{
  font-family: sans-serif;
}

.step-form__item{
  >label {
    font-weight: bold;
    text-align: left;
    display: block;
    margin: 8px 0;
  }
  select{
    display: block;
  }
  input[type="text"]{
    display: block;
    border: 1px solid #ccc;
    padding: 4px;
    &.blank {
      display: inline;
      width: 100px;
      outline: none;
      border: unset;
      border-bottom: 1px solid #ccc;
      margin: 0 5px
    }
  }

}

.medication__item{
  padding: 12px 8px;
  border: 1px solid #ccc;
  margin-bottom: 12px;
  margin-top: 12px;
}

select {
  width: 200px;
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
.bold{
  font-weight: bold;
}

.flex{
  display: flex;
}
.step-list{
  width: 20%;
}
.my-4{
  margin-bottom: 4px;
  margin-top: 4px;
}
</style>
