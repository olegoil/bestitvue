<script setup>
    import axios from 'axios';
    import { ref } from 'vue';
    
    const loadingshow = ref(false);
    let items = ref([]);
    const userData = ref({
        authenticated: false,
        state: '',
        username: '',
        password: ''
    });
    const filters = ref({});
    const userLogged = ref(localStorage.getItem('loggedin'));

    let shopdata =  location.href.indexOf('localhost') !== -1 ? 'http://localhost:8085/shopdata.php' : './shopdata.php';
    let userdata =  location.href.indexOf('localhost') !== -1 ? 'http://localhost:8085/userdata.php' : './userdata.php';

    function loadShop() {
        loadingshow.value = true;
        const obj = new URLSearchParams(filters.value);
        axios.get(shopdata, {
            params: filters.value
        }).then(function(response){
            if(response.data) {
                loadingshow.value = false;
                // filtershow.value = false;
                items.value = response.data;
                if(response.data[0]) {
                    
                }
            }
        }).catch(function(error){
            loadingshow.value = false;
            console.error(error);
        });
    }

    function login() {
        let userdataparams = JSON.parse(JSON.stringify(userData.value));
        userdataparams.login = 1;
        axios.get(userdata, {
            params: userdataparams
        })
        .then(function(response){
            console.log(response.data);
            if(response.data) {
                if(response.data.id) {
                    localStorage.setItem('loggedin', userData.value.username);
                    userLogged.value = localStorage.getItem('loggedin');
                    userData.value.authenticated = true;
                    loadShop();
                }
                else {
                    userData.value.state = response.data;
                }
            }
        })
        .catch(function(error){
            console.error(error);
        });
    }

    function logout() {
        localStorage.removeItem('loggedin');
        userData.value.authenticated = false;
        userData.value.state = '';
        userData.value.username = '';
        userData.value.password = '';
        userLogged.value = false;
    }

    if(userLogged.value) {
        loadShop();
    }

    function recalcPrice(val) {
        if(typeof val == 'number') {
            return val/100;
        }
        return val;
    }

</script>

    <template>

        <div class="container" v-if="!userLogged">
            <div class="row">
                <div class="col-md-12">
                    <div class="card">
                        <div class="form-data">
                            <label class="form-label">Benutzername
                                <input autocomplete="off" type="text" v-model="userData.username" class="form-control">
                            </label>
                            <label class="form-label">Passwort
                                <input autocomplete="off" type="password" v-model="userData.password" class="form-control">
                            </label>
                            <div class="mb-3" v-if="userData.state"> 
                                <p class="text-danger">{{ userData.state }}</p> 
                            </div>
                            <div class="mb-3"> 
                                <button class="btn btn-success" type="button" v-on:click="login()">Einloggen</button> 
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="container" v-if="userLogged">

            <div class="row">
                <div class="col-sm">
                    <button type="button" v-on:click="logout()" style="float:right;" class="btn btn-danger">Ausloggen</button>
                </div>
            </div>
            
            <div class="row" v-for="item in items">
                <div class="col-sm">
                    <div class="card mb-3">
                        <div class="row g-0">
                            <div class="col-md-4">
                            <img :src="'items/'+item.image" class="img-fluid rounded-start" alt="...">
                            </div>
                            <div class="col-md-8">
                            <div class="card-body">
                                <h5 class="card-title">{{item.name}}</h5>
                                <p class="card-text">Art. {{item.id}}</p>
                                <p class="card-text">{{item.description}}</p>
                                <p class="card-text"><small class="text-body-secondary">{{ recalcPrice(item.price) }} €</small></p>
                            </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </template>

<style>
    @import 'bootstrap';
</style>