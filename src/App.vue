<template>
    <div :class="[{ flexStart: step === 1 }, 'wrapper']">
        <transition name="slide">
            <img src="./assets/logo.png" class="logo" v-if="step === 1">
        </transition>
        <transition name="fade">
            <HeroImage v-if="step === 0" />
        </transition>
        <Claim v-if="step === 0" />
        <SearchInput v-model="searchValue" @input="handleInput" :dark="step === 1" />
        <div class="loader" v-if="step === 1 && loading" />
        <div class="results">
            <Item v-for="item in results" :item="item" :key="item.data[0].nasa_id" @click.native="handleModalOpen(item)" />
        </div>

        <Modal v-if="modalOpen" :item="modalItem" @closeModal="modalOpen = false" />
   </div>
</template>

<script>
    import axios from 'axios';
    import debounce from 'lodash.debounce';
    import Claim from '@/components/Claim.vue';
    import SearchInput from '@/components/SearchInput.vue';
    import HeroImage from '@/components/HeroImage.vue';
    import Item from '@/components/Item.vue';
    import Modal from "@/components/Modal.vue";

    const API = 'https://images-api.nasa.gov/search';

    export default {
        name: "App",
        components: {
            Modal,
            Claim,
            SearchInput,
            HeroImage,
            Item,
        },
        data () {
            return {
                modalOpen: false,
                modalItem: null,
                loading: false,
                step: 0,
                searchValue: '',
                results: [],
            };
        },
        methods: {
            handleModalOpen(item) {
                this.modalOpen = true;
                this.modalItem = item;
            },
            // eslint-disable-next-line
            handleInput: debounce(function() {
                this.loading = true;

                axios.get(`${API}?q=${this.searchValue}&media_type=image`)
                    .then((response) => {
                        this.results = response.data.collection.items;
                        this.loading = false;
                        this.step = 1;
                    })
                    .catch((error) => {
                        console.log(error);
                    });
            }, 800),
        },
    };
</script>

<style lang="scss">
    @import url('https://fonts.googleapis.com/css?family=Montserrat:300,400,600,800');
    * {
        box-sizing: border-box;
    }
    body {
        font-family: Montserrat, sans-serif;
        font-weight: 300;
        margin: 0;
        padding: 0;
        -webkit-font-smoothing: antialiased;
    }

    .fade-enter-active, .fade-leave-active {
      transition: opacity 1s ease;
    }

    .fade-enter, .fade-leave-to {
      opacity: 0;
    }

    .slide-enter-active, .slide-leave-active {
      transition: margin-top .3s ease;
    }

    .slide-enter, .slide-leave-to {
      margin-top: -50px;
    }

    .wrapper {
        margin: 0;
        position: relative;
        padding: 30px;
        width: 100%;
        min-height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;

        &.flexStart {
          justify-content: flex-start;
        }
    }

    .loader {
        margin-top: 100px;
        display: inline-block;
        width: 64px;
        height: 64px;
    }
    .loader:after {
        content: " ";
        display: block;
        width: 46px;
        height: 46px;
        margin: 1px;
        border-radius: 50%;
        border: 5px solid #1e3d4a;
        border-color: #1e3d4a transparent #1e3d4a transparent;
        animation: loading 1.2s linear infinite;
    }
    @keyframes loading {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    .logo {
        position: absolute;
        top: 30px;
    }

    .results {
        margin-top: 50px;
        display: grid;
        grid-template-columns: 1fr 1fr;
        grid-gap: 15px;

        @media (min-width: 768px) {
          width: 90%;
          grid-template-columns: 1fr 1fr 1fr;
        }
    }
</style>