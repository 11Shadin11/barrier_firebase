<template lang="pug">
  #app
    .form(v-if="showForm")
      <input ref="q" placeholder="Имя" type="text">
      <input ref="w" placeholder="Фамилия" type="text">
      <input ref="e" placeholder="Отчество" type="text">
      <input ref="r" placeholder="Марка и модель авто" type="text">
      <input ref="t" placeholder="Регистрационный знак авто" type="text">
      <input ref="y" placeholder="Номер телефона" type="text">
      <input ref="u" placeholder="Номер квартиры" type="text">
      <select ref="i" placeholder="Статус жильца" type="text">
        <option value="Владелец" selected>Владелец</option>
        <option value="Арендатор">Арендатор</option>
      </select>

      button.addUser(@click="addNewMessage" fab) Добавить
    .addUser(v-if="!showForm" @click="showForm = true") Добавить пользователья
    .allUsers
      .content-users(v-for="user in allUsers")
        .users
          b {{user.lastName}} {{user.firstName}} {{user.middleName}}
          b {{user.status}}
          b {{user.carInfo}}
          b {{user.carRegNumber}}
          b № Кв: {{user.appartmentNumber}}
          b Тел. : {{user.phoneNuber}}
    
</template>

<script>
import {getFirestore, onSnapshot, collection, addDoc, query} from 'firebase/firestore';
import { initializeApp } from "firebase/app";
import {ref,onUnmounted} from 'vue';

const firebaseConfig = {
  apiKey: "AIzaSyDGiuTr7FlQRZXuYaR4aqNq2QWi0IQYhKY",
  authDomain: "barrier-5acd6.firebaseapp.com",
  projectId: "barrier-5acd6",
  storageBucket: "barrier-5acd6.appspot.com",
  messagingSenderId: "604370511919",
  appId: "1:604370511919:web:8cb4cf34cb69895e903eda",
  measurementId: "G-BBZDHDYXR9"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
export default {
  name: 'App',

  data() {
    return {
      showForm: false,
      allUsers:ref([])
    }
  },

  methods: {

    addNewMessage:function() {
      addDoc(collection(db,'residents'),{
        firstName: this.$refs.q.value,
        lastName: this.$refs.w.value,
        middleName: this.$refs.e.value,
        carInfo: this.$refs.r.value,
        carRegNumber: this.$refs.t.value,
        phoneNuber: this.$refs.y.value,
        appartmentNumber: this.$refs.u.value,
        status: this.$refs.i.value,        
      });

      console.log(this.allUsers);
    },

  },

  mounted() {
    const latestQuery = query(collection(db,"residents"));
    const livemessages = onSnapshot(latestQuery,(snapshot)=>{
      this.allUsers = snapshot.docs.map((doc) => {
        console.log(doc);
        return {
          appartmentNumber: doc._document.data.value.mapValue.fields.appartmentNumber.stringValue,
          carInfo: doc._document.data.value.mapValue.fields.carInfo.stringValue,
          carRegNumber: doc._document.data.value.mapValue.fields.carRegNumber.stringValue,
          firstName: doc._document.data.value.mapValue.fields.firstName.stringValue,
          lastName: doc._document.data.value.mapValue.fields.lastName.stringValue,
          middleName: doc._document.data.value.mapValue.fields.middleName.stringValue,
          phoneNuber: doc._document.data.value.mapValue.fields.phoneNuber.stringValue,
          status: doc._document.data.value.mapValue.fields.status
        .stringValue}
      });
    });
    onUnmounted(livemessages)

    console.log(this.allUsers);
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
}
.content-users{
  display:flex;
  justify-content:center;
}
.users b {
  margin-top: auto;
  margin-bottom: auto;
  color: #000;
}
.users{
  cursor: pointer;
  display:flex;
  flex-wrap: wrap;
  justify-content:space-between;
  padding:10px;
  margin: 10px;
  width: 80%;
  background: rgb(156, 156, 156);
  border-radius:20px;
  height: 50px;
}
.users:hover{
  box-shadow: 0 5px 20px 2px #000;
}
.addUser{
  position: fixed;
  z-index: 100000;
}
.addUser {
  width: max-content;
  height: 40px;
  border: 1px solid #004790;
  border-radius: 5px;
  padding: 5px 15px;
  color: #004790;
  background: transparent;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
.form {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
</style>