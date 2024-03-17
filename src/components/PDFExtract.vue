<script setup>
import { ref } from 'vue';
import modal_Update from './modal_Update.vue';

const textareaInput = ref('');
const textareaOutput = ref('');
const strInvoice = ref('');
const strContract = ref('');
const strBuyer = ref('');

const words = ref([]);
const editIndex = ref(null);
const editedWord = ref('');
const orderPartNo = ref('');
const description = ref('');
const showModal = ref(false);

const showProcessButton = ref(false);
const showProcess = ref(false);

const showNotification = ref(false);
const notificationMessage = ref('');

const getText = () => {
  let strAll;
  let strBase = '';

  if(textareaInput.value == ''){
    Notify("There is no data to process.");
    return;
  }

  strBase = textareaInput.value.replace(/[\r\n]+/g, '|');

  strInvoice.value = extractBetween(strBase, 'Invoice No:|', '|Invoice Date:');
  strContract.value = extractBetween(strBase, 'Contract No : ', ' Warehouse:');
  strBuyer.value = extractBetween(strBase, 'Buyers Order No ', '|Country of Origin :');

  splitAndEdit(extractBetween(strBase, 'Amount|( JPY )|', 'Sub Total JPY'));
  // strAll = getItems(extractBetween(strBase, 'Amount|( JPY )|', 'Sub Total JPY')) + '\n';

  textareaOutput.value = strAll;

  //textareaOutput.value = strInvoice.value + '--'+ strContract.value +'--'+ strBuyer.value;

  showProcessButton.value = true;
};

const clearText = () => {
  textareaInput.value = '';

  words.value.splice(0);
  editedWord.value = '';
  description.value = '';
  orderPartNo.value = '';

  showProcessButton.value = false;
  showProcess.value = false;
};

const extractBetween = (thisText, startChar, endChar) => {
  const startPosition = thisText.indexOf(startChar);
  const endPosition = thisText.indexOf(endChar);

  if (startPosition > -1 && endPosition > -1) {
    return thisText.substring(startPosition + startChar.length, endPosition);
  } else {
    return startChar + 'Not found';
  }
};

const getItems = () => {
  showProcess.value = true;
  // let arrayList = [];
  let result = '';

  // arrayList = this_text.split('|');

  words.value.forEach((i) => {
    if (i === '') {
      return;
    }

    let resultItem = '';

    let arrayItem = [];
    arrayItem = i.split(' ');

    resultItem += strBuyer.value + '\t';
    resultItem += strContract.value + '\t';
    resultItem += strInvoice.value + '\t';

    switch (arrayItem.length) {
      case 7:
        resultItem += arrayItem[1] + '\t';
        resultItem += arrayItem[2].split('||').join(' ') + '\t'; //desc // fixed
        resultItem += arrayItem[3] + '\t';
        resultItem += '\t'; //for hidden column;
        resultItem += arrayItem[5] + '\t';
        resultItem += arrayItem[6] + '\n'; 

        break;
      default:
        resultItem = 'Error' + '\n';
        break;
    }
    result += resultItem;
  });

  textareaOutput.value = result;

  // return result;
};

const splitAndEdit = (this_text) => {
  words.value = this_text.split('|').filter(x => x !== '');
};

const countSpaces = (string) => {
  return string.split(' ').length - 1;
};

const editWord = (index) => {
  if (words.value[index].includes('|')) {
    editedWord.value = words.value[index].split('||').join(' ');
  }
  else {
    editedWord.value = words.value[index];
  }

  editIndex.value = index;
  description.value = '';
  orderPartNo.value = '';
  showModal.value = true;
};

const saveEdit = () => {
  if (description.value != '') {
    editedWord.value = editedWord.value.replace(description.value, description.value.replace(/ /g, '||'))
  }
  if (orderPartNo.value != '') {
    editedWord.value = editedWord.value.replace(' ' + orderPartNo.value, '(' + orderPartNo.value + ')')
  }

  words.value[editIndex.value] = editedWord.value;
  showModal.value = false;
  editIndex.value = null;
};

const closeModal = () => {
  showModal.value = false;
  editIndex.value = null; // Reset editIndex when modal is closed
};

const copyToClipboard = () => {
  navigator.clipboard.writeText(textareaOutput.value);
  Notify("Text copied successfully!");
};

const Notify = (message) => {
  showNotification.value = true;
  notificationMessage.value = message;
  setTimeout(() => {
    showNotification.value = false;
  }, 2000); // Hide notification after 2 seconds
}
</script>

<template>
  <header>

  </header>

  <main>
    <div>
      <h1>PDF Data Converter</h1>
    </div>
    <div>
      <textarea class="text-area" v-model="textareaInput" wrap="off" placeholder="Paste data for processing"></textarea>
    </div>

    <div>
      <button @click="getText">Process</button>
      <button @click="clearText">Clear</button>

    </div>
    <!-- Modal -->
    <div class="modal" v-if="showModal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>

        <label for="editedWord">Item:</label>
        <input type="text" v-model="editedWord" @keyup.enter="saveEdit">

        <label for="description">Description (DETECT):</label>
        <input type="text" id="description" v-model="description">
        <label for="orderPartNo">Ordered Part No (DETECT):</label>
        <input type="text" id="orderPartNo" v-model="orderPartNo">


        <button class="save-button" @click="saveEdit">Save</button>
      </div>
    </div>

    <div v-for="(word, index) in words" :key="index" :class="{ 'red-background': countSpaces(word) !== 6 }"
      class="word-container">
      <span v-if="editIndex === null || editIndex !== index">{{ word }}</span>
      <button @click="editWord(index)">Edit</button>
    </div>

    <button @click="getItems" v-show="showProcessButton">Get Excel Code</button>
    <div>
      <textarea v-model="textareaOutput" class="text-area" wrap="off" placeholder="Processed data will show here"
        v-show="showProcess"></textarea>

      <button @click="copyToClipboard" v-show="showProcess">Copy to Clipboard</button>
    </div>

    <!-- Notification -->
    <div class="notification" v-show="showNotification">{{ notificationMessage }}</div>

  </main>
</template>

<style scoped>
.modal {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #000000;
  padding: 20px;
  border: 1px solid #888;
  width: 50%;
  position: relative;
  z-index: 2;
  /* Ensure the modal content is on top of the overlay */
  display: flex;
  flex-direction: column;
}

.modal-content label {
  display: block;
}

.modal-content input[type="text"] {
  width: 100%;
  margin-bottom: 10px;
}

.modal-content .close {
  position: absolute;
  top: 0px;
  right: 10px;
  cursor: pointer;
}

.modal-content .save-button {
  align-self: center;
}

.red-background {
  background-color: red;
}

.word-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
  margin-bottom: 10px;
}

.text-area {
  min-width: 400px;
  width: 100%;
  height: 200px;
  margin-top: 10px;
}

.notification {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 63, 0, 0.8);
  color: #fff;
  padding: 10px 20px;
  border-radius: 5px;
  z-index: 3;
}
</style>
