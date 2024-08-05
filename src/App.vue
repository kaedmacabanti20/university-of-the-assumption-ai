<script>
import axios from 'axios';
export default {
  name : 'Chatbox',
  data() {
    return {
      uiData: {}, conversation:[],
      baseUrl: 'https://uagpt-private-server.vercel.app/',localUrl: 'http://localhost:3000/',
      initialMessage: `Hello! You can ask me questions about the university. I can only answer one question per minute because of my current settings.`, Name: '', input: '', profileImageSrc: '',
      isLoading: false, error: null,message: null, messageUser: null,isChatboxHidden: true, showChatMessage: false,
     };
  },
  mounted() {
    axios.get(this.baseUrl+'api/getNameFromMongoDB')
    .then(response => {
      this.Name = response.data.Name;
    })
    .catch(error => {
      console.error('Error fetching Name:', error);
      console.log('Response status:', error.response ? error.response.status : 'N/A');
      console.log('Response data:', error.response ? error.response.data : 'N/A');
    });

    // chat message
    setTimeout(this.showChatFor5Seconds, 8000);    4
    axios.get(this.baseUrl+'api/getImageFromMongoDB')
      .then(response => {
          const imageName = response.data.Image || 'stacy.png';
          const imageSrc = `/${imageName}`;
          this.loadImage(imageSrc);
      })
      .catch(error => {
          console.error('Error fetching image:', error);
      });
    },
  methods: {
    loadImage(src) {
      this.profileImageSrc = src;
    },
    deleteAll() {
        this.saveToLocalStorage();
        this.conversation = []; 
        this.initialMessage = ''; 
      },
    saveToLocalStorage() { 
      window.localStorage.setItem('conversation', JSON.stringify(this.conversation));
    },
    loadFromLocalStorage() {
      const storedconversation = window.localStorage.getItem('conversation');
      if (storedconversation) {
        this.conversation = JSON.parse(storedconversation);
      }
    },
 
    async fetchData() {
      this.isLoading = true;
  
      const options = {
        method: "POST",
        body: JSON.stringify({
          messages: this.input
        }), 
        headers: {
          "Content-Type": "application/json"
        }
      };
      try {
        const response = await fetch(this.baseUrl+'completions', options);
        const data = await response.json();
        const conMessage = data.choices[0].message;

        console.log("Testing")

        this.messageUser = { role: "user", content: this.input};
        this.message = { role: conMessage.role, content: conMessage.content};
        this.input = ' '
        this.conversation.push(this.messageUser, this.message);
        
 
        console.log(this.conversation) 
      } catch (error) {
        this.error = 'An error occurred while fetching data.';
      } finally {
        this.isLoading = false;
      }
    },
    toggleChatbox() {this.isChatboxHidden = !this.isChatboxHidden;},
    showChatbox() {this.isChatboxHidden = false;},
    showChatFor5Seconds() {this.showChatMessage = true;setTimeout(() => {this.showChatMessage = false;}, 5000);},
  },

};
</script>

<template>
  <div>
    <section class="chat-box" :style="{ display: isChatboxHidden ? 'none' : 'block' }">
      <div class="main-container">
        <ul :class="[profileImageSrc === '/UA-Logo.png' ? 'header-ua-logo' : 'header']">
          <li><img :src="profileImageSrc" alt="" :class="[profileImageSrc === '/UA-Logo.png' ? ' header-icon-ua-logo' : 'header-icon']"/> <span :class="[profileImageSrc === '/UA-Logo.png' ? ' name-ua-logo' : 'name']"><b>{{Name}}</b></span></li> 
          <li><i class="fa-solid fa-repeat refresh-icon"  @click="deleteAll"></i><i class="fa-solid fa-xmark x-icon" @click="toggleChatbox"></i></li>
        </ul>
        <div class="main-message-container">
          <div class="message-container-initial" v-if="initialMessage.trim() !== ''">
            <div class="bot-message">
              <span class="message" v-html="initialMessage"></span>
            </div>
          </div>
          <div v-for="(message, index) in conversation" :key="index" class="message-container">
            <div :class="[message.role === 'user' ? ' user-message' : 'bot-message']">
              <span class="message" v-html="message.content"></span>
            </div>
          </div>
          <div v-if="isLoading" class="loading">
            <div class="loading-box"><span class="dot"></span><span class="dot"></span><span class="dot"></span></div>
          </div>
        </div>
        <hr>
        <div class="footer">
          <span>
            <input type="text" v-model="input" @keyup.enter="fetchData" placeholder="Input question here">
            <button type="submit" @click="fetchData">
              <i class="fa-solid fa-paper-plane send-icon"></i>
            </button>
          </span>
        </div>
      </div>
    </section>
  
    <div v-if="isChatboxHidden" @click="showChatbox" class="show-chat-btn">
      <section v-if="showChatMessage" class="chat-btn-text">
        <span>Hello I'm {{ Name }}! you can ask me if you have any questions</span> 
        <div class="chat-arrow"></div>
       </section>
      <!-- <section class="chat-btn-img" > -->
      <section :class="[profileImageSrc === '/UA-Logo.png' ? 'chat-btn-img-ua-logo' : 'chat-btn-img']">
        <img :src="profileImageSrc" alt="">
      </section>
     </div>
  </div>
</template>

<style>

.chat-btn-text[style*="none"] {display: none;}
.show-chat-btn {position:fixed;bottom: 25px; right: 25px;border: none; cursor: pointer; height: 75px;display: flex; justify-content: center; align-items: center;}
.chat-btn-text{height: 100%;  margin: .5rem;display: flex; align-items: center;}
.chat-btn-text span{background-color: white;border-radius: 5px; height: 100%;display: flex; justify-content: center; align-items: center;padding: 1rem;}
.chat-arrow{width: 5px; height: 5px;border-left: 10px solid transparent; border-right: 10px solid transparent; border-bottom: 20px solid white; transform: rotate(90deg);}
.chat-btn-img{height: 65px; width: 65px;background-color: white;padding:10px; border-radius: 50%;box-shadow: 0 0 2px rgba(37, 37, 37, 0.5);display: flex; align-items: center; justify-content: center;}
.chat-btn-img img{height: 55px;}
.chat-btn-img-ua-logo img{height: 65px;width: 65px;}
/* CHATBOX  */
.chat-box {position: fixed; bottom: 0; right: 15px; width: 320px; height: 450px;box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); color: white; border-top-left-radius: 0.5rem; border-top-right-radius: 0.5rem;}
.main-container{border-top-left-radius: 0.5rem; border-top-right-radius: 0.5rem;background-color: white; display: flex; flex-direction: column; height: 100%; overflow-x: hidden;}
/* CHAT HEADER */
.header{height: 38.5px;}
.header-ua-logo{height: 52px;}
.main-container .header{display: flex; justify-content: space-between; align-items: center;border-top-left-radius: 0.5rem; border-top-right-radius: 0.5rem; background-color: #003075;padding: 0 .8rem 0 .8rem;}
.main-container .header-ua-logo{display: flex; justify-content: space-between; align-items: center;border-top-left-radius: 0.5rem; border-top-right-radius: 0.5rem; background-color: #003075;padding: 0 .8rem 0 .8rem;}
.main-container .header li{display: flex; align-items: center;}
.main-container .header-ua-logo li{display: flex; align-items: center;}
.header-icon {width: 100px; height: auto;position: absolute;margin: -82px 0 0 -10px;}
.header-icon-ua-logo {width: 40px; position: relative; margin: 0;}
.name-ua-logo {margin-left: 10px; font-size: 1.2rem;}
.name {margin-left: 95px;}
.refresh-icon{font-size: .9rem; padding: 11px 0; margin: 0 .8rem;}
.x-icon{padding: 11px 0; font-size: 1.1rem;}
/* MESSAGES */
.message-container-initial {margin-bottom: 1rem; margin-top:.6rem;}
.message-container {margin-bottom: 1rem;}
.user-message{display: flex; justify-content:flex-end; width: 100%;}
.user-message .message {background-color: #0084ff; padding: 0.5rem; border-radius: 0.5rem; color: white;}
.message{font-size: 15px; margin: 0 10px;}
.bot-message{display: flex; justify-content: flex-start; width: 100%;}
.bot-message .message {background-color: #e4e6eb;padding: 0.5rem;border-radius: 0.5rem;color: black; }
.main-message-container {display: flex; flex-direction: column; flex-grow: 1; padding: 0.4rem; overflow-y: auto;}
/* FOOTER */
.footer {padding: 0.5rem; display: flex;}
.footer span{background-color: #f3f3f5; width: 100%;}
.footer span input{background-color: transparent;width: 85%; padding: 0.5rem; font-size: 15px;border: none; outline: none; box-shadow: none;  color: #000; placeholder: #888; }
.footer span button{ width: 15%;}
.send-icon{ color: grey; }
/* LOADING BOX  */
.loading{color:black; margin: 10px;}
.loading-box {background-color: #f0f0f0;overflow: hidden; width: 50px;height: 30px;display: flex; align-items: center; justify-content: center;border: 1px solid #ccc;border-radius: 25px; border: none;}
.dot-container {display: flex; align-items: center; justify-content: center;}
.dot {width: 5px; height: 5px;background-color: #8c9095; border-radius: 50%; margin: 0 3px; animation: jump 1s infinite alternate;}
.dot:nth-child(2) {animation-delay: 0.2s;}
.dot:nth-child(3) {animation-delay: 0.4s;}
@keyframes jump {0%, 100% {transform: translateY(0);}50% {transform: translateY(-5px);}}
</style>
