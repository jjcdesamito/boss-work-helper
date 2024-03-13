<script>
  export default {
  props: {
    word: {
      type: String,
      required: true,
    },
  },
};
</script>

<template>
 <div v-if="showModal" class="modal">
    <div class="modal-content">
      <span class="close" @click="closeModal">&times;</span>
      <input type="text" v-model="editedWord" @keyup.enter="saveEdit">
      <button @click="saveEdit">Save</button>
    </div>
  </div>


    <div class="modal">
      <input type="text" v-bind:value="word" @input="$emit('update:word', $event.target.value)" />
      <button @click="$emit('close')">Close</button>
      <button @click="$emit('data-updated', word)">Save</button>
    </div>

</template>

<style>
 /* Modal styles */
 .modal {
      display: none; /* Hidden by default */
      position: fixed; /* Stay in place */
      z-index: 1; /* Sit on top */
      left: 0;
      top: 0;
      width: 100%; /* Full width */
      height: 100%; /* Full height */
      overflow: auto; /* Enable scroll if needed */
      background-color: rgb(0,0,0); /* Fallback color */
      background-color: rgba(0,0,0,0.4); /* Black w/ opacity */
    }
    
    /* Modal content */
    .modal-content {
      background-color: #fefefe;
      margin: 15% auto; /* 15% from the top and centered */
      padding: 20px;
      border: 1px solid #888;
      width: 80%; /* Could be more or less, depending on screen size */
    }
    
    /* Close button */
    .close {
      color: #aaa;
      float: right;
      font-size: 28px;
      font-weight: bold;
    }
    
    .close:hover,
    .close:focus {
      color: black;
      text-decoration: none;
      cursor: pointer;
    }
</style>