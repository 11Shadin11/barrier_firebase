<template lang="pug">
  #app
    .form(v-if="showForm")
      
      .d-flex
        select( style="width: 80%" ref="r" placeholder="Марка и модель авто")
          option( v-for="car in cars" :value="car.auto_model") {{car.auto_model}}
        button.addUser(style="margin: 10px; width: 20%" @click="showAddAuto = true") +
      
      .d-flex(v-if="showAddAuto")
        select( style="width: 80%" ref="auto" placeholder="Марка и модель авто")
          option( v-for="car in allCars" :value="car.auto_model") {{car.auto_model}}
        button.addUser(style="margin: 10px; width: 20%" @click="addNewAuto()") Добавить новый автомобиль

      select( ref="color" placeholder="Цвет Авто")
        option( value="" selected) Цвет авто
        option( value="Красный") Красный
        option( value="Зеленный") Зеленный
        option( value="Желтый") Желтый
      
      select(ref="type" placeholder="Тип кузова авто")
        option( value="" selected) Выберите тип кузова авто
        option( value="Седан") Седан
        option( value="Купе") Купе
        option( value="Хачбэк") Хачбэк
        option( value="Лифтбэк") Лифтбэк
        option( value="Кроссовер") Кроссовер
        option( value="Универсал") Универсал
        option( value="Внедорожник") Внедорожник

      input( ref="w" placeholder="Фамилия" type="text")
      input( ref="q" placeholder="Имя" type="text")
      input( ref="e" placeholder="Отчество" type="text")
      
      
      input( ref="t" placeholder="Регистрационный знак авто" type="text")
      input( ref="y" placeholder="Номер телефона" type="text")
      input( ref="u" placeholder="Номер квартиры" type="text")
      
      select( ref="i" placeholder="Статус жильца")
        option( value="Владелец" selected) Владелец
        option( value="Арендатор") Арендатор

      .d-flex
        button.addUser(@click="addNewUser" fab) Добавить
        button.addUser(@click="showForm = false" fab) Отменить
    .d-flex(v-if="!showForm")
      button.addUser( @click="showForm = true") Добавить нового пользователья
      button.addUser( @click="showArchive = !showArchive") {{!showArchive ? "Показать архивированных пользователей" : "Скрыть архивированных пользователей" }}
      input(style="margin: 0 10px; border-radius: 5px" type="date" v-model="dateFilter")

    .allUsers
      .content-users(v-for="user in returnUsers")
        .users
          b {{user.lastName}} {{user.firstName}} {{user.middleName}}
          b {{user.status}}
          b {{user.carInfo}}
          b {{user.typeCar}}
          b {{user.colorAuto}}
          b {{user.carRegNumber}}
          b № Кв: {{user.appartmentNumber}}
          b Тел. : {{user.phoneNuber}}
          b {{showArchive ? "Дата архивации" : "Дата добавления"}} : {{user.startDate}}
        button.addUser(v-if="!showArchive" @click="archivedUset(user)") Архивировать
      .content-users(v-if="showArchive && !archived.length" style="padding-top: 20px") Нет архивированных пользователей
</template>

<script>
import {getFirestore, onSnapshot, collection, doc, addDoc, deleteDoc, query} from 'firebase/firestore';
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
      dateFilter: null,
      showArchive: false,
      showAddAuto: false,
      allUsers:ref([]),
      archived:ref([]),
      allCars:ref([]),
      cars:ref([])
    }
  },

  computed: {
    returnUsers() {
      if(this.dateFilter) {
        let filter = []
        let allUserList = [...this.allUsers, ...this.archived]
        
        console.log(allUserList);

        allUserList.map(el => {
          if(el.state == "1") {
            if(Date.parse(new Date(el.startDate).toDateString()) <= Date.parse(new Date(this.dateFilter).toDateString())) filter.push(el)
          } 
          else if (el.state == "0") {
            if(Date.parse(new Date(el.startDate).toDateString()) >= Date.parse(new Date(this.dateFilter).toDateString())) filter.push(el)
          }
        })

        console.log(filter);
        return filter
      }

      if(this.showArchive) return this.archived

      return this.allUsers
    }
  },

  methods: {
   
    addNewAuto() {
      let obj = {auto_model: this.$refs.auto.value}
      
      this.cars.forEach(el => {
        if(el.auto_model == this.$refs.auto.value) deleteDoc(doc(db,'auto_list', el.id))
      });
    
      addDoc(collection(db,'auto_list'), obj)
      this.showAddAuto = false
    },

    addNewUser() {
      addDoc(collection(db,'residents'),{
        firstName: this.$refs.q.value,
        lastName: this.$refs.w.value,
        middleName: this.$refs.e.value,
        carInfo: this.$refs.r.value,
        carRegNumber: this.$refs.t.value,
        phoneNuber: this.$refs.y.value,
        appartmentNumber: this.$refs.u.value,
        status: this.$refs.i.value,
        colorAuto: this.$refs.color.value,
        typeCar: this.$refs.type.value,
        startDate: new Date(),
        state: 1  
      });

      this.showForm = false
    },

    archivedUset(user) {
      deleteDoc(doc(db,'residents', user.id))
      user.startDate = new Date()
      user.state = 0
      addDoc(collection(db,'archive_list'), user);
    }

  },

  mounted() {
    const latestQueryArchive = query(collection(db,"archive_list"));
    const livemessagesArchive = onSnapshot(latestQueryArchive,(snapshot)=>{
      this.archived = snapshot.docs.map((doc) => {
        return {
          appartmentNumber: doc._document.data.value.mapValue.fields.appartmentNumber.stringValue,
          carInfo: doc._document.data.value.mapValue.fields.carInfo.stringValue,
          carRegNumber: doc._document.data.value.mapValue.fields.carRegNumber.stringValue,
          firstName: doc._document.data.value.mapValue.fields.firstName.stringValue,
          lastName: doc._document.data.value.mapValue.fields.lastName.stringValue,
          middleName: doc._document.data.value.mapValue.fields.middleName.stringValue,
          phoneNuber: doc._document.data.value.mapValue.fields.phoneNuber.stringValue,
          colorAuto: doc._document.data.value.mapValue.fields.colorAuto.stringValue,
          typeCar: doc._document.data.value.mapValue.fields.typeCar.stringValue,
          status: doc._document.data.value.mapValue.fields.status.stringValue,
          startDate: doc._document.data.value.mapValue.fields.startDate.timestampValue,
          state: doc._document.data.value.mapValue.fields.state.integerValue
        }
      });
    });
    
    onUnmounted(livemessagesArchive)

    const latestQueryAllAuto = query(collection(db,"all_auto_list"));
    const livemessagesAllAuto = onSnapshot(latestQueryAllAuto,(snapshot)=>{
      this.allCars = snapshot.docs.map((doc) => {
        return {
          id: doc.id,
          auto_model: doc._document.data.value.mapValue.fields.auto_model.stringValue,
        }
      });
    });
    
    onUnmounted(livemessagesAllAuto)

    const latestQueryAuto = query(collection(db,"auto_list"));
    const livemessagesAuto = onSnapshot(latestQueryAuto,(snapshot)=>{
      this.cars = snapshot.docs.map((doc) => {
        return {
          id: doc.id,
          auto_model: doc._document.data.value.mapValue.fields.auto_model.stringValue,
        }
      });
    });
    
    onUnmounted(livemessagesAuto)

    const latestQuery = query(collection(db,"residents"));
    const livemessages = onSnapshot(latestQuery,(snapshot)=>{
      this.allUsers = snapshot.docs.map((doc) => {
        return {
          id: doc.id,
          appartmentNumber: doc._document.data.value.mapValue.fields.appartmentNumber.stringValue,
          carInfo: doc._document.data.value.mapValue.fields.carInfo.stringValue,
          carRegNumber: doc._document.data.value.mapValue.fields.carRegNumber.stringValue,
          firstName: doc._document.data.value.mapValue.fields.firstName.stringValue,
          lastName: doc._document.data.value.mapValue.fields.lastName.stringValue,
          middleName: doc._document.data.value.mapValue.fields.middleName.stringValue,
          phoneNuber: doc._document.data.value.mapValue.fields.phoneNuber.stringValue,
          colorAuto: doc._document.data.value.mapValue.fields.colorAuto.stringValue,
          typeCar: doc._document.data.value.mapValue.fields.typeCar.stringValue,
          startDate: doc._document.data.value.mapValue.fields.startDate.timestampValue,
          status: doc._document.data.value.mapValue.fields.status.stringValue,
          state: doc._document.data.value.mapValue.fields.state.integerValue
        }
      });
    });
    onUnmounted(livemessages)
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
  margin: 60px 10% 0 10%;
  display: flex;
  flex-direction: column;
}
.content-users{
  display:flex;
  justify-content:center;
  align-items: center;
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
  margin: 10px 0;
  width: 100%;
  background: rgb(156, 156, 156);
  border-radius:5px;
  height: 50px;
}
.users:hover{
  box-shadow: 0 0 10px #000;
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

input, select {
  padding: 10px;
  margin: 10px 0;
}
.d-flex {
  display: flex;
}

button:nth-child(2) {
  margin-left: 10px;
}
</style>