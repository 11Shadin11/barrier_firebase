<template lang="pug">
  #app(:class="{'center' : showForm}")
    .form(v-if="showForm")
      
      h2(style="font-weigth: 900; color: #fff") Добавление нового пользователья

      .d-flex
        select( style="width: 80%" ref="r" placeholder="Марка и модель авто")
          option( v-for="car in cars" :value="car.auto_model") {{car.auto_model}}
        button.addUser(style="margin: 10px 0 10px 10px; width: 20%" @click="showAddAuto = true") +
      
      .d-flex(v-if="showAddAuto")
        select( style="width: 80%" ref="auto" placeholder="Марка и модель авто")
          option( v-for="car in allCars" :value="car.auto_model") {{car.auto_model}}
        button.addUser(style="margin: 10px 0 10px 10px; width: 20%" @click="addNewAuto()") Добавить новый автомобиль

      select( ref="color" placeholder="Цвет Авто")
        option( value="" selected) Выбери цвет авто
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

      input( ref="t" placeholder="Регистрационный знак авто" type="text")

      input( ref="w" placeholder="Фамилия" type="text")
      input( ref="q" placeholder="Имя" type="text")
      input( ref="e" placeholder="Отчество" type="text")
      
      
      input#phone( ref="y" @input="phone()" placeholder="Номер телефона" type="tel")
      input( ref="u" placeholder="Номер квартиры" type="text")
      
      select( ref="i" placeholder="Статус жильца")
        option( value="" selected) Выберите статус жильца
        option( value="Владелец") Владелец
        option( value="Арендатор") Арендатор

      .d-flex
        button.addUser(@click="addNewUser" fab) Добавить
        button.addUser(@click="showForm = false" fab) Отменить
    
    .d-flex(v-if="!showForm" style="padding: 50px 10% 0 10%")
      button.addUser( @click="showForm = true") Добавить нового пользователья
      button.addUser( @click="showArchive = !showArchive" :class="{'active' : showArchive }") {{!showArchive ? "Показать архивированных пользователей" : "Скрыть архивированных пользователей" }}
      input(style="margin: 0 10px; border-radius: 5px" type="date" v-model="dateFilter")

    .allUsers(v-if="!showForm")
      h2(style="font-weigth: 900; padding: 50px 0; color:#fff") {{!showArchive ? "Активные пользователи" : "Архивированные пользователи" }} | {{returnUsers.length}}
      .content-users(v-for="user in returnUsers")
        .photo(style="width:330px; height: 200px")
          img(src="@/assets/man.jpg" width="100%" height="100%" )
        .info
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
          button.addUser( style="margin-left: 0" v-if="!showArchive" @click="archivedUset(user)") Архивировать
      .content-users(v-if="showArchive && !archived.length" style="padding-top: 20px") Нет архивированных пользователей
</template>

<script>
import {getFirestore, onSnapshot, collection, doc, addDoc, deleteDoc, query} from 'firebase/firestore';
import { initializeApp } from "firebase/app";
import {ref,onUnmounted} from 'vue';
import IMask from 'imask'

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

    phone() {
      let element = document.getElementById('phone');
      let maskOptions = {
        mask: '+7(000)000-00-00',
        lazy: false
      } 
      element.innerText = new IMask(element, maskOptions);
    },

    carRegNum() {
      let element = document.getElementById('carRegNum');
      let maskOptions = {
        mask: '*000** 00',
        lazy: false
      } 
      element.innerText = new IMask(element, maskOptions);
    },

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
          startDate: new Date(doc._document.data.value.mapValue.fields.startDate.timestampValue,).toDateString(),
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
          startDate: new Date(doc._document.data.value.mapValue.fields.startDate.timestampValue,).toDateString(),
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
*{
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  overflow-y: auto;
  color: #2c3e50;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;
  background: #292929;
  box-shadow: 0 -200px 100px -120px rgba(255, 60, 0, 0.47) inset;
  animation: background 6s infinite alternate;
}

@keyframes background {
  50% {
    background: #292929;;
    box-shadow: 0 -200px 100px -100px #1d1d1d inset;
  }
}
.content-users{
  display:flex;
  justify-content:center;
  align-items: center;
  height: 200px;
  margin-bottom: 20px;
}
.users b {
  margin-top: auto;
  margin-bottom: auto;
  color: #000;
}

.info {
  width: 100%;
  height: 100%;
  padding: 0 10px;
}
.photo {
  padding: 10px;
}

.photo img {
  border-radius: 10px;
}
.users{
  cursor: pointer;
  display:flex;
  flex-wrap: wrap;
  justify-content:space-between;
  padding:10px;
  margin: 10px 0;
  /* width: 100%; */
  /* background: rgb(156, 156, 156); */
  background: rgba(250, 100, 0, .9); 
  border-radius:5px;
  height: 50%;
}
.users:hover{
  box-shadow: 0 0 10px #646464;
}
.addUser {
  width: max-content;
  height: 40px;
  border: 1px solid rgb(147, 34, 0);
  border-radius: 5px;
  padding: 5px 15px;
  color: #ffffff;
  background: rgba(255, 96, 0, .2);;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
.form {
  width: 80%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 15px;
  border: 2px solid;
  border-radius: 5px;
  box-shadow: 0 0 10px #000;
  background: zerkal;
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
.active {
  box-shadow: 5px 5px 5px #646464;
  transform: translateY(-2px) scale(1.01);
}
.allUsers {
  height: 100%;
  padding: 0 10%;
}
.center {
  justify-content: center;
  align-items: center;
}
</style>