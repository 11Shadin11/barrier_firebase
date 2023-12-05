<template>
  <div>
    <h1>Add New Data</h1>
    <form>
      <label for="dataInput">New Data:</label>
      <input type="text" id="dataInput" v-model="newData" />
      <button type="submit" @click="addData">Add Data</button>
    </form>
  </div>
</template>

<script>
import firebase from 'firebase/app';
import 'firebase/database';

export default {
  data() {
    return {
      newData: ''
    };
  },
  methods: {
    addData() {
      // Получаем ссылку на корень базы данных
      const db = firebase.database();
      const dataRef = db.ref('data');

      // Добавляем новое поле в базу данных
      dataRef.push({
        value: this.newData
      });

      // Очищаем поле ввода после добавления данных
      this.newData = '';
    }
  }
};
</script>
