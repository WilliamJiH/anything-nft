<template>
    <div id="app" v-loading='!testnet'>
        <router-view />
        <h1 v-if='!testnet' id='testnet-err'>
            Please install ConfluxPortal and switch to testnet, then refresh.
        </h1>
    </div>
</template>

<script>
const { abi: inftyNftAbi } = require("./assets/InftyNft.json");
const { abi: stakeAbi } = require("./assets/StakingForINFT.json");
const { abi: raffleAbi } = require("./assets/Raffle.json");

import { eventBus } from "./main";
// const { Conflux } = require('js-conflux-sdk')

export default {
    name: "App",
    data: () => ({
        testnet: true
    }),
    created() {
        document.title = "Infty Marketplace";
        const minterContract = window.confluxJS.Contract({
            abi: inftyNftAbi,
            address: this.$store.getters.getMinterAddress,
        });
        this.$store.commit("setMinterContract", minterContract);

        const stakeContract = window.confluxJS.Contract({
            abi: stakeAbi,
            address: this.$store.getters.getStakeAddress,
        });
        this.$store.commit("setStakeContract", stakeContract);

        const raffleContract = window.confluxJS.Contract({
            abi: raffleAbi,
            address: this.$store.getters.getRaffleAddress,
        });
        this.$store.commit("setRaffleContract", raffleContract);

        eventBus.$on("App.notifyWIP", () => {
            this.$notify.info({
                title: "Info",
                message: 'This functionality is work in progress.',
                duration: 3000,
            });
        });

        eventBus.$on("App.notifyLoading", (msg) => {
            this.$notify({
                title: "Notification",
                dangerouslyUseHTMLString: true,
                message: `<div style="display:flex; align-items: center;"> <div class="loader"></div><div style="display:inline">${msg}</div></div>`,
                duration: 0,
            });
        });
    },
    mounted() {
        window.setTimeout(() => {
            if (!window.conflux) {
                this.testnet = false
                return
            }
            if (window.conflux.networkVersion != 1) {
                this.testnet = false
            }
        },1000)
    },
    beforeDestroy() {
        eventBus.$off("App.notifyWIP");
        eventBus.$off("App.notifyLoading");
    },
};
</script>
<style scoped>
.err {
    position: fixed;
    top: 0;
}
</style>
<style>

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
}

.flex-wrapper {
    display: flex;
    min-height: 100vh;
    flex-direction: column;
    justify-content: space-between;
}

.flex-wrapper-row {
    display: flex;
    width: 100%;
    flex-direction: row;
    justify-content: space-around;
}

.loader {
  display: inline-block;
  border: 2px solid #f3f3f3; /* Light grey */
  border-top: 2px solid #3498db; /* Blue */
  border-radius: 50%;
  width: 20px;
  height: 20px;
  margin-right: 10px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

#testnet-err {
    position: absolute;
    top: 40vh;
    left: 50%;
    transform: translate(-50%,-50%);
    z-index: 10000;
}
.back-btn {
    position: absolute;
    top: 150px;
    left: -10px;
    width: 60px;
    border-radius: 20%;
    background-color: rgb(72, 83, 87);
    border: unset;
    box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.2);
}
</style>
