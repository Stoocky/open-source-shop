<template>
    <div class="shopContainer" id="Mainbody" style="width: 50%; height: 75%">
        <div class="shopBackground">
            <div class="shopWrapper">
                <div class="shopItem" v-for="(shopItem, index) in filteredItems" :key="index">
                    <div class="item" v-if="ShopSystem.ShopItems">
                        <div class="image" v-if="shopItem.image !== 'crate'">
                            <img :src="ResolvePath(`${shopItem.image}`)" id="Images" alt="shopBg" />
                        </div>
                        <div class="image" v-else>
                            <img :src="ResolvePath(`../../assets/icons/crate.png`)" id="Images" alt="shopBg" />
                        </div>
                        <div class="descriptions">
                            <span>{{ shopItem.name }}</span
                            ><br /><br />
                        </div>
                        <div class="inputButtons">
                            <span>{{ addCommas(shopItem.price) }}$</span><br /><br />
                            <input
                                type="number"
                                placeholder="1"
                                v-model="selectedAmount[index]"
                                @change="updateInput(this)"
                                maxlength="3"
                                oninput="this.value = Math.abs(this.value)"
                            />
                            <template v-if="shopAcceptsCard">
                                <Button class="buyButton" :color="buttonColor" @click="changeMoneyType">
                                    <Icon :size="16" :icon="moneyButton" />
                                </Button>
                            </template>
                            <template v-else>
                                <Button class="buyButton" color="grey" :disable="true">
                                    <Icon :size="16" icon="icon-money1" />
                                </Button>
                            </template>
                            <Button
                                class="buyButton"
                                :color="buttonColor"
                                :flatten="false"
                                :padding="2"
                                @click="buyShopItem(index)"
                                >{{ buttonText }}</Button
                            >
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="decoration" style="position: relative">
            <div class="logo"></div>
            <div class="descriptions">
                <p style="text-align: center">{{ shopName }}</p>
            </div>
            <div class="search-bar">
                <input type="text" v-model="search" placeholder="" required />
                <div class="search-icon"></div>
            </div>
            <Button color="red" class="btn-close" @click="closePage()"> schließen </Button>
        </div>
    </div>
</template>

<script lang="ts">
// @ts-nocheck
import { defineComponent } from 'vue';
import { ShopEvents } from '../shared/enums/ShopEvents';

import Button from '@components/Button.vue';
import Icon from '@components/Icon.vue';
import ResolvePath from '@utility/pathResolver';
import WebViewEvents from '../../../../../src-webviews/src/utility/webViewEvents';

// DEBUGGING
const SHOP = [
    { name: 'Northern Haze Seeds', dbName: 'Northern Haze Seeds', price: 250, image: 'burger' },
    { name: 'Bread', dbName: 'Bread', price: 350, image: 'bread' },
    { name: 'Burger', dbName: 'burger', price: 450, image: 'burger' },
    { name: 'Bread', price: 550, image: 'bread' },
    { name: 'Burger', price: 650, image: 'burger' },
    { name: 'Bread', price: 750, image: 'bread' },
];

const ComponentName = 'OSS_ShopUI';
export default defineComponent({
    name: ComponentName,
    components: {
        Button,
        Icon,
    },
    data() {
        return {
            shopType: 'buy',
            buttonText: 'Kaufen!',
            buttonColor: 'green',
            search: '',
            selectedAmount: [],
            keyword: '',
            shopLogo: './osslogo.png',
            shopName: '',
            shopAcceptsCard: false,
            usingCash: true,
            moneyButton: 'icon-money1',
            ShopSystem: {
                ShopItems: [],
            },
        };
    },
    mounted() {
        if ('alt' in window) {
            WebViewEvents.emitClient(`${ComponentName}:Ready`);
            WebViewEvents.on(ShopEvents.SET_ITEMS, this.fillShopItems);
        } else {
            this.fillShopItems(SHOP, 'buy', 'Test', true);
        }
    },
    computed: {
        filteredItems() {
            return this.ShopSystem.ShopItems.filter((ShopItem: { name: string }, index: number) =>
                ShopItem.name.toLowerCase().includes(this.search.toLowerCase()),
            );
        },
        cssVars() {
            return {
                '--img': this.shopLogo,
            };
        },
    },
    methods: {
        updateInput(input: number) {
            if (input.value.length > input.maxLength) {
                input.value = input.value.slice(0, input.maxLength);
            }

            if (input.value < 1) {
                input.value = 1;
            }
        },
        addCommas(nStr: string) {
            const [x1, x2] = nStr.toString().split('.');
            const rgx = /(\d+)(\d{3})/;
            let formatted = x1.replace(rgx, '$1.$2');
            if (x2) formatted += `.${x2}`;
            return formatted;
        },
        fillShopItems(shopItems, type: string, shopName: string, acceptsCard: boolean) {
            this.ShopSystem = {
                ShopItems: shopItems,
                ShopName: shopName,
                AcceptsCard: acceptsCard,
            };
            if (type === 'sell') {
                this.buttonText = 'Verkaufen';
                this.buttonColor = 'red';
                this.shopType = 'sell';
            } else {
                this.buttonText = 'Kaufen';
                this.buttonColor = 'green';
                this.shopType = 'buy';
            }
        },
        buyShopItem(index: number) {
            const { filteredItems, shopType, usingCash } = this;
            const selectedAmount = this.selectedAmount[index] ?? 1;
            WebViewEvents.emitServer(ShopEvents.HANDLE_SHOP, filteredItems[index], selectedAmount, shopType, usingCash);
        },
        changeMoneyType() {
            if (this.shopAcceptsCard) {
                this.usingCash = !this.usingCash;
                this.moneyButton = this.usingCash ? 'icon-money1' : 'icon-bank';
            }
        },
        closePage() {
            window.alt?.WebViewEvents.emitClient(ShopEvents.CLOSE_SHOP);
        },
        ResolvePath,
    },
});
</script>

<style scoped>
/* SHOPWRAPPER - DO NOT MODIFY */
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
.shopWrapper {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
}
.shopContainer {
    width: 60vw;
    padding: 0px;
    height: auto;
}
.shopItem {
    position: relative;
    color: white;
    user-select: none;
    height: auto;
    margin-bottom: 10px;
}
.shopItem .item {
    color: white;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
        'Open Sans', 'Helvetica Neue', sans-serif;
    font-weight: 700;
    font-size: 1em;
    background: #000;
    border: 1px solid rgba(29, 126, 171, 0.5);
    padding: 1rem;
    margin: 1rem;
}
.shopItem .image {
    max-height: 128px;
}
.shopItem .descriptions {
    position: relative;
    margin-top: 1.5vh;
    text-overflow: ellipsis;
    word-wrap: break-word;
    overflow: hidden;
    max-height: 4.4em;
    line-height: 1.4em;
    text-align: center;
    padding-left: 5%;
    padding-right: 5%;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
        'Open Sans', 'Helvetica Neue', sans-serif;
}
.shopBackground {
    position: absolute;
    background: rgba(0, 0, 0, 0.95);
    left: 10vw;
    top: 10vh;
    height: 80vh;
    width: 50vw;
    text-align: center;
    user-select: none;
    margin: 0 auto;
    border-top-left-radius: 15px;
    border-bottom-left-radius: 15px;
    overflow-y: scroll;
    overflow-x: hidden;
}
#Images {
    width: 128px;
    height: 128px;
    max-height: 128px;
    padding-top: 20px;
}
.decoration {
    width: 15vw;
    height: 75vh;
    background: linear-gradient(180deg, rgba(35, 39, 42, 1) 0%, rgba(44, 47, 51, 0.7511379551820728) 100%),
        url('OSS_shopUI.jpg');
    float: right;
    background-size: cover;
    background-position: center;
    padding-top: 0;
}
.decoration input {
    width: 100%;
    height: 100%;
    border: none;
    background: transparent;
    color: white;
    font-size: 1.5em;
    font-weight: bold;
    text-align: center;
    outline: none;
}
.logo {
    margin-left: 10%;
    width: 80%;
    height: auto;
    min-height: 222px;
    background: url('./osslogo.png') no-repeat;
    background-size: contain;
    margin-top: 10%;
}
.shopItem input {
    background-color: rgba(43, 112, 158, 0.788);
    width: 80%;
    max-width: 80%;
    color: white;
    widows: 100%;
    text-align: center;
    user-select: none;
    height: 2.2vh;
    border: 0px;
    border-radius: 5px;
    border-color: white;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
        'Open Sans', 'Helvetica Neue', sans-serif;
    font-size: 1.1em;
    font-weight: bolder;
    margin-bottom: 1vh;
}
.inputButtons {
    position: relative;
    top: -0.5vh;
    align-items: center;
}
.buyButton {
    border-radius: 0px;
    border: 0px;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
        'Open Sans', 'Helvetica Neue', sans-serif;
}
.btn-grad {
    width: 75%;
    left: 12.5%;
    margin-top: 5%;
    padding: 25px 45px;
    text-align: center;
    text-transform: uppercase;
    transition: 0.5s;
    background: rgb(0, 0, 0);
    background-size: 100% auto;
    border: 1px solid rgb(20, 171, 218);
    border-left: 0px;
    border-right: 0px;
    border-radius: 50px;
}
.btn-grad:hover {
    background-position: right center; /* change the direction of the change here */
    color: #fff;
    text-decoration: none;
}
.btn-close {
    max-width: 15vw;
    float: right;
    position: absolute;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.75);
    border: 0px;
    width: 100%;
}
/* width */
::-webkit-scrollbar {
    width: 10px;
    border-radius: 25px;
}
/* Track */
::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.35);
}
/* Handle */
::-webkit-scrollbar-thumb {
    background: rgba(255, 0, 0, 0.35);
}
/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
    background: rgba(255, 0, 0, 0.35);
}
::placeholder {
    color: rgba(255, 255, 255, 0.363);
    opacity: 1;
}
.search-bar {
    height: auto;
    width: auto;
    position: absolute;
    display: inline-block;
    margin-top: 50px;
    top: 20%;
    left: 50%;
    transform: translate(-50%, -50%);
    box-sizing: border-box;
}
.search-bar input {
    height: 44px;
    width: 44px;
    padding: 10px 20px;
    box-sizing: border-box;
    font-size: 18px;
    border: 2px solid transparent;
    border-radius: 5px;
    cursor: pointer;
    font-weight: 100;
    background-color: transparent;
    transition: all 0.5s ease-out;
    color: transparent;
}
.search-bar input::-moz-placeholder {
    color: transparent;
}
.search-bar input:-ms-input-placeholder {
    color: transparent;
}
.search-bar input::placeholder {
    color: transparent;
}
.search-bar input:invalid {
    box-shadow: none;
}
.search-bar input:hover {
    border: 2px solid #fff;
}
.search-bar input:focus,
.search-bar input:valid {
    width: 12vw;
    border: 2px solid rgb(0, 0, 0);
    outline: none;
    cursor: auto;
    background-color: rgba(0, 0, 0, 0.548);
    color: white;
}
.search-bar input:focus::-moz-placeholder,
.search-bar input:valid::-moz-placeholder {
    color: #999;
}
.search-bar input:focus:-ms-input-placeholder,
.search-bar input:valid:-ms-input-placeholder {
    color: #999;
}
.search-bar input:focus::placeholder,
.search-bar input:valid::placeholder {
    color: #999;
}
.search-bar input:focus + .search-icon,
.search-bar input:valid + .search-icon {
    z-index: 0;
    border-color: #ccc;
    right: 20px;
}
.search-bar input:focus + .search-icon:after,
.search-bar input:valid + .search-icon:after {
    background-color: #ccc;
}
.search-icon {
    display: inline-block;
    height: 20px;
    width: 20px;
    border-radius: 50%;
    border: 2px solid #fff;
    position: absolute;
    right: 12px;
    top: 8px;
    z-index: -1;
}
.search-icon:after {
    content: '';
    position: absolute;
    top: 19.07px;
    left: 17.07px;
    transform: rotate(45deg);
    height: 2px;
    width: 10px;
    background-color: #fff;
    border-radius: 10px;
}
</style>
