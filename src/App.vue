<script setup>
import { ref, watch } from 'vue';

const formData = ref([]);

const groupedData = ref([{ userName: '', phoneNum: '', email: '' },{ userName: '', phoneNum: '', email: '' }]);

const checkBoxes = ref(['Angular', 'Vuejs', 'ReactJs']);
const selectFields = ref([
  'New York', 'Los Angeles', 'Chicago', 'Houston', 'Phoenix',
  'Philadelphia', 'San Antonio', 'San Diego', 'Dallas',
]);

const selectedCity = ref('');
const selectedDate = ref('');
const selectedFile = ref(null);
const selectedCheckboxes = ref([]);
const fileType = ref('')

const errors = ref([{ nameError: '', emailError: '', phoneNumError: '' }]);
const additionalErrors = ref({
  cityError: '',
  dateError: '',
  fileError: '',
  checkboxError: ''
});

const addField = () => {
  groupedData.value.push({ userName: '', phoneNum: '', email: '' });
  errors.value.push({ nameError: '', emailError: '', phoneNumError: '' });
};

const removeField = (index) => {
  groupedData.value.splice(index, 1);
  errors.value.splice(index, 1);
};

watch(groupedData, (newValue) => {
  newValue.forEach(val => {
    const digits = val.phoneNum.replace(/[^\d]/g, '');
    if (digits) {
      val.phoneNum = digits.replace(/(\d{3})(\d{3})(\d{4})/, "($1) $2-$3");
    } else {
      val.phoneNum = digits;
    }
    val.userName  = val.userName.replace(/[^a-zA-Z]/g, '')
  });
}, { deep: true });
const handleSubmit = () => {
  let isValid = true;


  additionalErrors.value = {
    cityError: '',
    dateError: '',
    fileError: '',
    checkboxError: ''
  };

  if (!selectedCity.value) {
    additionalErrors.value.cityError = 'City is required';
    isValid = false;
  }

  if (!selectedDate.value) {
    additionalErrors.value.dateError = 'Birth date is required';
    isValid = false;
  }

  if (!selectedFile.value) {
    additionalErrors.value.fileError = 'File upload is required';
    isValid = false;
  }


  if (selectedCheckboxes.value.length === 0) {
    additionalErrors.value.checkboxError = 'At least one skill must be selected';
    isValid = false;
  }

  groupedData.value.forEach((grp, index) => {
    const errorObj = { nameError: '', emailError: '', phoneNumError: '' };
    if (!grp.userName) {
      errorObj.nameError = 'Name is required';
      isValid = false;
    }
    if (!grp.phoneNum) {
      errorObj.phoneNumError = 'Phone number is required';
      isValid = false;
    }
    if (!grp.email) {
      errorObj.emailError = "Email is required";
      isValid = false;
    }
    errors.value[index] = errorObj;
  });

  if (isValid) {
    formData.value.push({
      city: selectedCity.value,
      birthDate: selectedDate.value,
      fileName: selectedFile.value.name,
      skills: [...selectedCheckboxes.value],
      users: groupedData.value.map(grp => ({
        name: grp.userName,
        email: grp.email,
        phone: grp.phoneNum
      }))
    });

    errors.value = [{ nameError: '', emailError: '', phoneNumError: '' }];
    selectedCity.value = '';
    selectedDate.value = '';
    selectedFile.value = null;
    selectedCheckboxes.value = [];
  }
};

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    const validTypes = ['application/pdf', 'image/jpeg'];
    if (validTypes.includes(file.type)) {
      selectedFile.value = file;
      additionalErrors.value.fileError = ''; 
    } else {
      selectedFile.value = null;
      additionalErrors.value.fileError = 'Only PDF or JPEG files are allowed';
    }
  }
};

</script>
<template>
<div class="mainWrapper">


  <div style="margin-bottom: 20px; border: 1px solid #ccc; padding: 15px; border-radius: 8px;" class="inputWrapper">

    <div v-for="(grp, index) in groupedData" :key="index">
      <div style="margin-bottom: 10px;">
        <label>Phone Number:</label><br />
        <input 
          placeholder="Phone Number"  
          v-model="grp.phoneNum"
          maxLength="10"
          style="width: calc(100% - 15px); padding: 8px; margin-top: 5px;" 
        />
        <span v-if="errors[index]?.phoneNumError" style="color: red;">{{ errors[index].phoneNumError }}</span>
      </div>

      <div style="margin-bottom: 10px;">
        <label>Email:</label><br />
        <input 
          type="text" 
          placeholder="Email" 
          v-model="grp.email"
          pattern='[A-Za-z\\s]*'
          style="width: calc(100% - 15px); padding: 8px; margin-top: 5px;"
        />
        <span v-if="errors[index]?.emailError" style="color: red;">{{ errors[index].emailError }}</span>
      </div>

      <div style="margin-bottom: 10px;">
        <label>Name:</label><br />
        <input 
          v-model="grp.userName" 
          placeholder="Full Name" 
          style="width: calc(100% - 15px); padding: 8px; margin-top: 5px;"
        />
        <span v-if="errors[index]?.nameError" style="color: red;">{{ errors[index].nameError }}</span>
      </div>

      <div style="display: flex; gap: 10px;">
        <button @click="addField" style="padding: 6px 12px;">Add Field</button>
        <button 
          v-if="groupedData.length > 2" 
          @click="removeField(index)" 
          style="padding: 6px 12px; background-color: #f44336; color: white;"
        >
          Remove Field
        </button>
      </div>
    </div>

    <div style="display:flex; padding-top: 12px; gap:5px; flex-wrap: wrap;">
      <div v-for="check in checkBoxes" :key="check">
        <input type="checkbox" :value="check" v-model="selectedCheckboxes" />
        {{check}}
      </div>
    </div>
    <span v-if="additionalErrors.checkboxError" style="color:red">{{ additionalErrors.checkboxError }}</span>

    <div style="display:flex; padding-top: 12px; gap:10px; flex-wrap: wrap;">
      <input type="file" @change="handleFileChange" accept="image/jpeg, .pdf"/>
      <span v-if="additionalErrors.fileError" style="color:red">{{ additionalErrors.fileError }}</span>

      <select v-model="selectedCity">
        <option value="">Select a City</option>
        <option v-for="city in selectFields" :value="city" :key="city">{{ city }}</option>
      </select>
      <span v-if="additionalErrors.cityError" style="color:red">{{ additionalErrors.cityError }}</span>
    </div>

    <div style="margin-top: 12px;">
      <input type="date" v-model="selectedDate" />
      <span v-if="additionalErrors.dateError" style="color:red">{{ additionalErrors.dateError }}</span>
    </div>

    <div style="margin-top: 30px;">
      <button 
        @click="handleSubmit" 
        style="padding: 10px 20px; background-color: #4CAF50; color: white; font-weight: bold;"
      >
        Submit
      </button>
    </div>

  </div>
   <table v-if="formData.length" style="width: 100%; margin-top: 20px; border-collapse: collapse;" class="tableContentHolder">
  <thead>
    <tr>
      <th style="border: 1px solid #ccc; padding: 8px;">City</th>
      <th style="border: 1px solid #ccc; padding: 8px;">Birth Date</th>
      <th style="border: 1px solid #ccc; padding: 8px;">File</th>
      <th style="border: 1px solid #ccc; padding: 8px;">Skills</th>
      <th style="border: 1px solid #ccc; padding: 8px;">Users</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(data, i) in formData" :key="i">
      <td style="border: 1px solid #ccc; padding: 8px;">{{ data.city }}</td>
      <td style="border: 1px solid #ccc; padding: 8px;">{{ data.birthDate }}</td>
      <td style="border: 1px solid #ccc; padding: 8px;">{{ data.fileName }}</td>
      <td style="border: 1px solid #ccc; padding: 8px;">{{ data.skills.join(', ') }}</td>
      <td style="border: 1px solid #ccc; padding: 8px;">
        <table style="width: 100%; border-collapse: collapse;">
          <thead>
            <tr>
              <th style="border: 1px solid #ccc; padding: 4px;">Name</th>
              <th style="border: 1px solid #ccc; padding: 4px;">Email</th>
              <th style="border: 1px solid #ccc; padding: 4px;">Phone</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(user, idx) in data.users" :key="idx">
              <td style="border: 1px solid #ccc; padding: 4px;">{{ user.name }}</td>
              <td style="border: 1px solid #ccc; padding: 4px;">{{ user.email }}</td>
              <td style="border: 1px solid #ccc; padding: 4px;">{{ user.phone }}</td>
            </tr>
          </tbody>
        </table>
      </td>
    </tr>
  </tbody>
</table>
</div>

</template>
