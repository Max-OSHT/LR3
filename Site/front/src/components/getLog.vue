-- Active: 1714724041901@@127.0.0.1@3306@lr2
<template>
    <header>
        <nav class="navbar navbar-expand-sm" @submit.prevent="">
            <div class="container mt-0 px-2">
                <div class="align-top" loading="lazy" id="label">
                    <a @click="logo" id="logo">Log Profile</a>
                    <!-- <router-link :to="{ name: 'inside' }" id="logo"> Log Profile </router-link> -->
                </div>
                <button class="navbar-toggler ms-auto" type="button" data-bs-toggle="collapse"
                    data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent"
                    aria-expanded="false" aria-label="Переключатель навигации">
                    <span>
                        <svg class="icon" width="30" height="25">
                            <use xlink:href="#align_text_distribute [#914]"></use>
                        </svg>
                    </span>
                </button>
                <div class="collapse navbar-collapse" id="navbarSupportedContent">
                    <ul class="d-flex justify-content-around navbar-nav px-auto w-100" id="navbar">
                        <div class="nav-item m-2 px-2">

                            <a @click="info">Info</a>
                            <!-- <router-link :to="{ name: 'info' }" @click="call"> Info </router-link> -->
                        </div>
                        <div class="nav-item m-2 px-2">
                            <a @click="warning">Warning</a>
                            <!-- <router-link :to="{ name: 'warning' }"> Warning </router-link> -->
                        </div>
                        <div class="nav-item m-2 px-2">
                            <a @click="error">Error</a>
                            <!-- <router-link :to="{ name: 'error' }"> Error </router-link> -->
                        </div>
                        <div class="nav-item m-2 px-2">
                            <a @click="critical">Critical</a>
                            <!-- <router-link :to="{ name: 'critical' }"> Critical </router-link> -->
                        </div>
                        <button v-if="isLoggedIn" type="button" id="Out" class="btn btn-outline-light"
                            @click="logout">Выйти</button>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="search">
        <form id="search">
            <div>
                <input type="text" class="form-control" v-model="searchQuery" placeholder="Поиск">
            </div>
            <div>
                <VueDatePicker v-model="date" :dark="true" range @update:model-value="getDate" />
            </div>
        </form>
    </div>
    <div class="list">
        <ul class="list_content" v-for="item in filteredData">
            <li v-for="el in item">
                <div class="main">
                    <div>
                        <p>{{ el.id }}</p>
                    </div>
                    <div><textarea cols="100">{{ el.data }}</textarea></div>
                </div>
            </li>
        </ul>
    </div>
</template>

<script>
import axios from 'axios'
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'
export default {
    components: {
        VueDatePicker
    },
    data() {
        return {
            searchQuery: '',
            log: [],
            date: "",
            date_on: '',
            date_two: '',
            isAuthorized: true,
            year: '',
            month:'',
            day:''
        };
    },
    methods: {
        logo(){
            this.searchQuery = '';
        },
        info(){
            this.searchQuery = '';
            this.searchQuery = 'info';
        },
        error(){
            this.searchQuery = '';
            this.searchQuery = 'error';
        },
        warning(){
            this.searchQuery = '';
            this.searchQuery = 'warning';
        },
        critical(){
            this.searchQuery = '';
            this.searchQuery = 'critical';
        },
        async getDate() {
            // console.log(this.date)

            // const date_on = `${year}-${month}-${day}`;

            if (this.date != undefined) {
                if (this.date[0] != null) {
                    // console.log(this.date[0]);
                    this.year = this.date[0].getFullYear();
                    this.month = this.date[0].getMonth() + 1;
                    this.day = this.date[0].getDay() + 5;
                    if(this.month < 10) this.month = "0".concat(this.month);
                    if(this.day < 10) this.day = "0".concat(this.day);  

                    this.date_on = `${this.year}-${this.month}-${this.day}`;
                    // console.log(this.date_on);
                }
                if (this.date[1] != null) {
                    this.year = this.date[1].getFullYear();
                    this.month = this.date[1].getMonth() + 1;
                    this.day = this.date[1].getDay() + 5;
                    if(this.month < 10) this.month = "0".concat(this.month);
                    if(this.day < 10) this.day = "0".concat(this.day);

                    this.date_two = `${this.year}-${this.month}-${this.day}`;

                    // this.date_two = this.date[1].toISOString().slice(0, 10);
                    // console.log(this.date_two);
                }
            }
            if (this.date == undefined) {
                this.date_on = "";
                this.date_two = "";
            }
        },
        async logout() {
            this.$store.dispatch('logout')
                .then(() => { this.$router.push({ name: 'home' }) })
        }

    },
    computed: {
        filteredData() {
            const searchQueryLower = this.searchQuery.toLowerCase();
            // const searchQueryUpper = this.searchQuery.toUpperCase();
            return this.log.map(item => {
                return item.filter(logItem => {                    
                    if ((logItem.data.toLowerCase().includes(searchQueryLower)) && (this.searchQuery != "")) {
                        // console.log(Object.values(logItem));
                        return logItem;
                    }
                    if(logItem.data.includes(this.date_on) && this.date_on != "")
                    {
                        // console.log(this.date_on, this.date_two);
                        return logItem;
                    }
                    if(logItem.data.includes(this.date_two) && this.date_two != "")
                    {
                        // console.log(this.date_on);
                        return logItem;
                    }
                    if ((this.date_on == "" || this.date_two == "") && this.searchQuery == "") {
                        // console.log(dt_one)
                        return item;
                    }
                })
            });

        },
        isLoggedIn : function() { 
            return this.$store.getters.isLoggedIn
        },
    },

    created() {
        axios.interceptors.response.use(undefined, function (err) {
            return new Promise((resolve, reject) => {
                if (err.status === 401 && err.config && !err.config.__isRetryRequest) {
                    this.$store.dispatch('logout')
                        .then(() => { this.$router.push({ name: 'home' }) })
                }
                throw err;
            });
        });

        axios.defaults.headers.common['Authorization'] = "Token " + localStorage.getItem('token')
        axios.get('http://localhost:8000/log/')
            .then((response) => {
                this.log = Object.values(response.data);
            })
            .catch(err => console.log(err))
    },

}
</script>

<style lang="scss" scoped>
header {
  background-color: #0c0c0c;
  padding: 20px 0;
  a {
    text-decoration: none;
    font-weight: 700;
    color: #ffffff;
    font-size: 25px;
  }
  .nav-item > a {
    text-decoration: none;
    font-weight: 700;
    color: #ffffff;
    font-size: 25px;
  }

  #logo{
    color: rgb(51, 182, 123);
    font-size: 30px;
  }

  button {
    margin-left: auto;
    border-width: 3px;
    border-color: rgb(51, 182, 123);
  }
  ul > div {
    border-color: rgb(51, 182, 123);
  }
  #search {
    padding: 0;
    margin: 0;
  }
  @media (max-width: 700px) {
    #label {
      display: none;
      visibility: hidden;
    }
    #navbarSupportedContent {
      width: 100%;
    }
  }
  @media (max-width: 575.98px) {
    #navbarSupportedContent {
      text-align: center;
    }
    #Out {
      width: 100%;
      margin-top: 10px;
    }
    #label {
      display: block;
      visibility: visible;
    }
  }
}

* {
    font-family: 'Roboto', sans-serif;
}

.list li {
    border: 2px solid rgb(51, 182, 123);
    border-radius: 5px;
    margin-bottom: 10px;
    padding: 10px;
    background-color: #0c0c0c;
}

.search {
    background-color: #0c0c0c;
}

#date {
    background-color: #0c0c0c;
    color: #ffffff;
}

#search {
    padding-bottom: 10px;
    width: calc(100% - 22%);
    margin: 0 auto;
    display: flex;
    justify-content: space-around;

}

#search>div {
    width: 100%;
}

#search>div:first-child {
    margin-right: 10px;
}

input {

    margin: 0 auto;
    border: 2px solid rgb(51, 182, 123);
    background-color: #0c0c0c;

}

.list_content {
    width: calc(100% - 22%);
    margin: 0 auto;
    list-style: none;
    padding: 0;
}

.list {
    background-color: #0c0c0c;
}

.main {
    display: flex;
    flex-direction: row;
    justify-content: space-evenly;
    align-items: center;
    border-radius: 5px;
}

p {
    color: #ffffff;
    margin-bottom: 0%;

}

textarea {
    width: calc(100%);
    border: #ffffff;
    background-color: #0c0c0c;
    color: #ffffff;
}

@media (max-width: 600px) {
    #search {
        display: block;
    }

    #search>div {
        margin-bottom: 10px;
    }
}


@media (max-width: 1200px) {
    .main {
        display: block;
    }



    .main>div {
        text-align: center;
    }

}
</style>
