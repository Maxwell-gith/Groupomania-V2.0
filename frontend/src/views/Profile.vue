<template>
    <section class="sectionProfile">
        <div class="sectionProfile__card">
            <h1 class="sectionProfile__card__title">Mon profil</h1>
            <div v-if="dataUser.image" class="sectionProfile__card__image">
                <img :src="dataUser.image" alt="Photo de profil">
            </div>
            <div v-else class="sectionProfile__card__image">
                <img src="../assets/profilepics.jpg" alt="Photo de profil">
            </div>
            <ul class="sectionProfile__card__content">
                <label v-if="mode == 'update'" for="fileInput" class="sectionProfile__card__content__actionButton primaryButton">Ajouter une image</label>
                <em v-if="urlImage">Votre nouvelle image: {{ this.urlImage }}</em>
                <input v-if="mode == 'update'" class= "sectionProfile__card__content__addFile" id="fileInput" type="file" @change="addImg()" ref="file" />
                <input type="text" v-if="mode == 'update'" v-model="dataUser.name" placeholder="Nom"> 
                <li class="sectionProfile__card__content__data" v-else>Nom : {{ dataUser.name }}</li>
                <input type="text" v-if="mode == 'update'" v-model="dataUser.firstname" placeholder="Prénom">
                <li class="sectionProfile__card__content__data" v-else>Prénom : {{ dataUser.firstname }}</li>
                <input type="email" v-if="mode == 'update'" v-model="dataUser.email" placeholder="Email"> 
                <li class="sectionProfile__card__content__data" v-else>Email : {{ dataUser.email }}</li> 
            </ul>
            <button class="sectionProfile__card__button primaryButton" v-if="mode == 'update'" @click="sendUpdate()">Sauvegarder</button>
            <button class="sectionProfile__card__button secondaryButton" v-if="mode == 'update'" @click="switchToRead(), getData()">Annuler</button>
            <div class="sectionProfile__card__buttonContainer" v-else>
                <button class="sectionProfile__card__buttonContainer__button primaryButton" @click="switchToUpdate()">Modifier</button>
                <button class="sectionProfile__card__buttonContainer__buttonDelete primaryButton" @click="confirmDelete()">Supprimer mon Compte</button>
            </div>
        </div>
    </section>
</template>

<script>
import axios from "axios";

export default {
    name: 'profile',
    props: {
        id: Number,
    },
    data() {
        return {
            token: "",
            userId: "",
            dataUser: [],
            mode: 'read',
            errorAlert: "",
            file: null,
            urlImage: "",
        };
    },
    methods: {
        getData() {
            let token = localStorage.getItem("token");
            let userId = localStorage.getItem("id");
            axios
                .get("http://localhost:3000/api/profile/" + userId, {
                    headers: { Authorization: "Bearer " + token},
                })
                .then((res) => {
                    console.log(res);
                    this.dataUser = res.data;
                    localStorage.setItem("image", res.data.image);
                    document.getElementById("fileInput").value='';
                    this.file = null;
                })
                .catch((error) => {
                    console.log({ error });
                });
        },
        
        sendUpdate() {
            let token = localStorage.getItem("token");
            let userId = localStorage.getItem("id");
            let data = new FormData();
            if (this.file !== null && document.getElementById("fileInput").value !='') {
                data.append("name", this.dataUser.name);
                data.append("firstname", this.dataUser.firstname);
                data.append("email", this.dataUser.email);
                data.append("image", this.file, this.file.name);
            } else {             
                data.append("name", this.dataUser.name);
                data.append("firstname", this.dataUser.firstname);
                data.append("email", this.dataUser.email);
            }
            axios
                .put("http://localhost:3000/api/profile/" + userId, data
                , {
                    headers: { Authorization: "Bearer " + token },
                })
                .then((res) => {
                    console.log(res);
                    this.getData();
                    this.switchToRead();
                    this.urlImage = "";
                })
                .catch((error) => {
                    console.log({ error });
                    alert(this.errorAlert = error.response.data.error);
                });
        },
        deleteAccount(){
            let token = localStorage.getItem("token");
            let userId = localStorage.getItem("id");
            axios
                .delete("http://localhost:3000/api/profile/" + userId, {
                    headers: { Authorization: "Bearer " + token },
                })
                .then((res) => {
                    console.log(res);
                    localStorage.clear();
                    this.$router.push("/");
                })
                .catch((error) => {
                    console.log({ error });
                });
        },
        confirmDelete() {
            if (confirm("Voulez-vous vraiment supprimer votre compte ?")) {
                this.deleteAccount();
            }
        },    
        addImg() {
            this.file = this.$refs.file.files[0];
            this.urlImage = document.getElementById("fileInput").value;
        },
        switchToUpdate() {
            this.mode = 'update';
        },
        switchToRead() {
            this.mode = 'read';
        },
    },
    mounted() {
        this.getData();
    },
}
</script>

<style lang="scss" scoped>
@import "@/assets/_shared.scss";

    .sectionProfile{
        min-height: 100vh;
        min-width: 100%;
        display: flex;
        justify-content: center;
        padding: 35px 0 35px 0;
        &__card{
            width: 80%;
            height: 100%;
            border-radius: 10px;
            background-color: $secondaryColor;
            padding: 35px 15px 35px 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            &__title{
                color: $primaryColor;
                margin-bottom: 35px;
            }
            &__image{
                width: 100px;
                height: 100px;
                border-radius: 50%;
                border: $primaryColor 3px solid;
                overflow: hidden;
                margin-bottom: 15px;
                img{
                    width: 100%;
                    height: 100%;
                    object-fit: cover;
                }
            }
            &__content{
                list-style: none;
                width: 100%;
                display: flex;
                flex-direction: column;
                justify-content: center;
                align-items: center;
                &__data{
                    margin-bottom: 15px;
                }
                &__actionButton{
                    width: 50%;
                    height: 40px;
                    margin-bottom: 25px;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                }
                &__addFile{
                    display: none;
                }
                em{
                    color: $tertiaryColor;
                    font-size: 11px;
                    font-weight: 700;
                    margin-bottom: 15px;
                }
            }
            &__button{
                width: 50%;
                height: 40px;
                margin-bottom: 25px;
            }
            &__buttonContainer{
                width: 100%;
                display: flex;
                flex-direction: column;
                align-items: center;
                &__button{
                    width: 50%;
                    height: 40px;
                    margin-bottom: 25px;
                }
                &__buttonDelete{
                    background-color: $primaryColor;
                    color: $bodyColor;
                    width: 50%;
                    height: 40px;
                    margin-bottom: 25px;
                }
            }
        }
    }

    input{
        width: 90%;
        height: 40px;
        border-radius: 5px;
        border: none;
        padding: 5px;
        margin-bottom: 25px;
        @include shadow;
    }

    @media only screen and (min-width: 750px) {
    .sectionProfile__card{
        width: 70%;
        margin-top: 65px;
    }
    }

    @media only screen and (min-width: 1024px) {
    .sectionProfile__card{
        width: 40%;
    }
    }
</style>