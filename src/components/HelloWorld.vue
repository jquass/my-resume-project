<template>
  <h2>Hello, World!</h2>

  <div>
    <h5>{{ helloMsg }}</h5>
  </div>

  <Transition name="input-transition" @after-leave="showButton">
    <div v-if="!inputDisabled">
      <div>
        <input @input="updateHelloMsgDebounce" v-model="helloInput" placeholder="Your Name" type="text" id="name"
               name="name"
               maxlength="20" :disabled="inputDisabled" autocomplete="off">
      </div>
    </div>
  </Transition>
  <Transition name="button-transition" @after-leave="showInput">
    <div v-if="!buttonDisabled">
      <button :disabled="buttonDisabled" @click="hideButton">Did I Get Your Name Wrong?</button>
      <div v-if="censored">
        <p>I had to censor part of your name. For more information on what words are censored, see <a
            href="https://www.purgomalum.com/index.html" target="_blank">PurgoMalum</a></p>
      </div>
      <div v-if="cat">
        <img id="cat" :src="image.url">
        <h5>Powered by <a href="https://thecatapi.com" target="_blank">The Cat API </a></h5>
        <button @click="updateCat">New Cat Please</button>
        <p>{{image.count}} Cats Fetched That I Can Remember</p>
      </div>
    </div>
  </Transition>
</template>

<script>

import debounce from "debounce";
import axios from 'axios';

export default {
  name: 'HelloWorld',
  data() {
    return {
      defaultHelloMsg: "Let me know your name for a personalized message!",
      helloMsg: null,
      helloInput: null,
      updateHelloMsgDebounce: null,
      inputDisabled: false,
      buttonDisabled: true,
      censored: false,
      cat: false,
      image: null
    }
  },
  methods: {
    async processHelloMsg() {
      this.hideInput();
      await Promise.all([this.updateHelloMessage(), this.checkForCat()]);
    },
    async updateHelloMessage() {
      let params = {}
      if (this.helloInput && this.helloInput.length > 0) {
        params.name = this.helloInput
      }
      let response = await axios.get('http://localhost:9000/hello-world', {
        params: params,
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        }
      });
      let data = response['data'];
      console.log(data);
      this.helloMsg = data["message"] + " I have made " + data["count"] + " greetings that I can remember!";
      this.censored = Boolean(data["censored"]);
    },
    async checkForCat() {
      this.cat = this.helloInput.toLowerCase() === 'cat';
      if (this.cat) {
       await this.updateCat();
      }
    },
    async updateCat() {
      let response = await axios.get('http://localhost:9000/the-cat-api', {
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        }
      });
      let data = response['data'];
      console.log(data);
      this.image = data;
    },
    showButton() {
      this.buttonDisabled = false;
    },
    hideButton() {
      this.buttonDisabled = true;
    },
    showInput() {
      this.initializeHelloMsg();
      this.helloInput = null;
      this.inputDisabled = false;
    },
    hideInput() {
      this.inputDisabled = true;
    },
    initializeHelloMsg() {
      this.helloMsg = this.defaultHelloMsg;
    }
  },
  created() {
    this.updateHelloMsgDebounce = debounce(this.processHelloMsg, 800);
    this.initializeHelloMsg();

  }
}
</script>

<style scoped>
h2 {
  margin: 40px 10px;
}

div {
  margin: 20px 20px;
}

#cat {
  max-width: 75%;
}

.input-transition-enter-active,
.input-transition-leave-active {
  transition: opacity 0.5s ease;
}

.input-transition-enter-from,
.input-transition-leave-to {
  opacity: 0;
}

.button-transition-enter-active,
.button-transition-leave-active {
  transition: opacity 0.5s ease;
}

.button-transition-enter-from,
.button-transition-leave-to {
  opacity: 0;
}

</style>
