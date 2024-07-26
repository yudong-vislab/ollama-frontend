<template>
  <div class="converse">
    <h2>Conversation</h2>
    <select v-model="selectedModel">
      <option value="llava:7b">llava 7b</option>
      <option value="llama3">llama3 8b</option>
    </select>
    <!-- <select llm-mode="respondemode">
      <option value="generate">generate</option>
      <option value="chat">chat</option>
    </select> -->
    <input v-model="userInput" placeholder="Ask a question" class="question-input" />
    <input type="file" @change="onFileChange" ref="fileInput" />
    <div v-if="imageUrl" class="images-preview">
      <img :src="imageUrl" alt="Image Preview" />
    </div>
    <button @click="askQuestion">Ask</button>
    <div class="conversation-container">
      <div v-for="(message, index) in conversation" :key="index" class="message">
        <div :class="{'user-message': message.role === 'user', 'bot-message': message.role !== 'user'}">
          <p class="message-content">
            <strong>{{ message.role === 'user' ? 'Question:' : 'Answer' }}</strong>
            {{ message.role === 'user' ? message.content : `(${message.time} sec): ${message.content}`}}
          </p>
          <p class="conclude">
            <strong>{{ 'conclude' }}</strong>
            {{ conclude }}
          </p>
          <div v-if="message.images" class="message-images">
            <img :src="message.images" alt="Message Image" />
          </div>
        </div>
      </div>
    </div>
    <div v-if="error" class="error">
      <p>{{ error }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Converse',
  data() {
    return {
      userInput: '',
      imageFile: null,
      imageUrl: '',
      conversation: [],
      error: null,
      responseTime: null,
      selectedModel: 'llava' ,
      conclude:'here is conclusion'
      // respondemode:'chat'
    };
  },
  methods: {
    onFileChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.imageFile = file;
        this.imageUrl = URL.createObjectURL(file);
      }
    },
    // async conclude(response){
    //   try {
    //     responsed = await axios.post('http://127.0.0.1:5000/api/generate', {
    //       model: this.selectedModel,
    //       prompt: "我将给你提供一段内容，这段内容是对一张图片的描述。请你用一两个词来总结这段文字，能够实现对这张未知图片的描述和标记。图片的文字描述如下："+response,
    //     });
    //   }catch (error) {
    //     this.error = `Error: ${error.message}`; // Remove the last user message in case of an error to maintain conversation integrity
    //   }
    //   const conclude = responsed.data.response;
    //   return conclude
    // },
    async askQuestion() {
    this.error = null;
    this.responseTime = null;

    const startTime = new Date(); // Record start time

    let imageBase64 = '';
    if (this.imageFile) {
      imageBase64 = await this.convertToBase64(this.imageFile);
    }
    const userMessage = {
      role: 'user',
      content: this.userInput,
      images: imageBase64 ? [imageBase64] : null,
      time: null // User messages have no time
    };
    this.conversation.push(userMessage);

    this.userInput = ''; // Clear input field
    this.imageFile = null;
    this.imageUrl = ''; // Clear image preview
    this.$refs.fileInput.value = ''; // Clear file input
    this.scrollToBottom(); // Scroll to the bottom of the conversation

    try {
      const response = await axios.post('http://127.0.0.1:5000/api/chat', {
        model: this.selectedModel,
        messages: this.conversation,
        // respondemode:this.respondemode
      });
    const endTime = new Date(); // Record end time
    this.responseTime = ((endTime - startTime) / 1000).toFixed(2); // Calculate response time in seconds
    this.conclude = response.data.message.conclude
    const botMessage = {
          role: 'assistant',
          content: response.data.message.content,
          images: response.data.message.image,
          time: this.responseTime, // Include time in bot messages
        };
    this.conversation.push(botMessage);
      } catch (error) {
        this.error = `Error: ${error.message}`;
        this.conversation.pop(); // Remove the last user message in case of an error to maintain conversation integrity
      }
    // try{
    //   const resp = this.conversation[-1];
    //   const respe=resp.content;
    //   const responsed = await axios.post('http://127.0.0.1:5000/api/generate', {
    //     model: this.selectedModel,
    //     prompt: "我将给你提供一段内容，这段内容是对一张图片的描述。请你用一两个词来总结这段文字，能够实现对这张未知图片的描述和标记。图片的文字描述如下："+respe,
    //   });
    //   this.conclude = responsed.data.conclude;
    // }catch (error) {
    //   this.error = `Error: ${error.message}`;}
    },

    convertToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
      });
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const container = this.$el.querySelector('.conversation-container');
        container.scrollTop = container.scrollHeight;
      });
    }
  }
};
</script>

<style scoped>
.converse {
  position: fixed;
  top: 10px;
  left: 10px;
  width: 400px; /* 增加宽度以适应滚动条 */
  height: 95vh; /* 设置高度以使其可滚动 */
  padding: 20px;
  background-color: #fff;
  border: 1px solid #ccc;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  overflow: hidden; /* 隐藏默认溢出 */
}

.question-input {
  width: 100%;
  padding: 5px;
  margin-bottom: 10px;
}

input[type="file"] {
  display: block;
  margin-bottom: 10px;
}

button {
  padding: 5px 10px;
  margin-bottom: 20px; /* 增加按钮与对话之间的间隔 */
}

.images-preview {
  max-width: 100%;
  max-height: 200px;
  overflow: hidden;
  margin-bottom: 10px; /* 增加图片预览与按钮之间的间隔 */
}

.images-preview img {
  width: 100%;
  height: auto;
  display: block;
}

.conversation-container {
  height: calc(100% - 120px); /* 留出输入框、按钮和间隔的空间 */
  overflow-y: auto; /* 垂直滚动 */
  padding-right: 10px; /* 防止内容被滚动条覆盖 */
}

.message {
  margin-bottom: 10px;
}

.user-message {
  text-align: left;
  background-color: #e6f7ff;
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 10px;
}

.bot-message {
  text-align: left;
  background-color: #f5f5f5;
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 10px;
}

.message-content {
  margin: 0;
}

.conclude{
  text-align: left;
  background-color: #f5f5f5;
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 10px;
}

.message-images img {
  max-width: 100%;
  max-height: 200px;
  display: block;
}

.error {
  color: red;
  text-align: left;
}
</style>
