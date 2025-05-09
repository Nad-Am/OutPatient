<template>
    <div class="chat-container">
      <div class="chat-header">
        <h2>医患交流</h2>
      </div>
      <div class="chat-messages" ref="chatMessages">
        <div v-for="(message, index) in messages" :key="index" :class="['message', message.sender]">
          <div class="message-avatar">
            <img :src="message.avatar" alt="头像" />
          </div>
          <div class="message-content">
            <div class="message-sender">{{ message.sender === 'patient' ? '患者' : '医生' }}</div>
            <div class="message-text">{{ message.text }}</div>
          </div>
        </div>
      </div>
      <div class="chat-input">
        <textarea
          v-model="newMessage"
          @keypress.enter="sendMessage"
          placeholder="输入消息..."
          class="textarea-field"
          rows="3"
        ></textarea>
        <button @click="sendMessage(diagId)" class="send-button">发送</button>
      </div>
    </div>
  </template>
  
  <script lang="ts" setup>
  import { ref, onMounted, nextTick, watch} from 'vue';
  import { useUserStore } from '@/stores/user';
  import { useComunicationStore } from '@/stores/comunication';
  import SockJS from 'sockjs-client';
  import { Client  } from '@stomp/stompjs';
  import { ElMessage } from 'element-plus';
  import { DoAxiosWithErro } from '@/api';

  const avatarUrl = ['/src/assets/me.png', '/src/assets/doctor.png'];
  const sender = ['patient', 'doctor'];

  const props = defineProps({
    diagId: {
      type: Number,
      required: true,
    },
  })

  const messages = ref([
    {
      sender: 'doctor',
      text: '您好！请问有什么可以帮助您的吗？',
      avatar: '/src/assets/doctor.png',
    },
  ]);
  
  const userStore = useUserStore();
  const comunicationStore = useComunicationStore();
  const newMessage = ref('');
  const chatMessages = ref<HTMLElement | null>(null);

  const [doctorId, patientId] = [userStore.userInfo.doctorId, userStore.userInfo.patientId];
  

  // 获取消息记录
  // const getHistory = (diagId: number) => {
  //   DoAxiosWithErro(`/medical/diagnoses/${diagId}/feedback`, 'get', {}, true).then((res) => {
  //     console.log(res);
  //     });
  // }

  const addMessage = (message: string,senderType: number) => {
    const newMessage = {
      sender: sender[senderType],
      text: message,
      avatar: avatarUrl[senderType],
    };
    messages.value.push(newMessage);
    scrollToBottom();
  };

  // 发送消息
  const sendMessage = (diagId: number = 1) => {
    if (newMessage.value.trim() === '') {
      ElMessage.warning('请输入消息内容');
      return;
    };

    DoAxiosWithErro(`/medical/diagnoses/${diagId}/feedback?content=${newMessage.value}`,'post',{},true).then((res) => {
      ElMessage.success('发送成功');
      addMessage(newMessage.value, +(!patientId));
      // 清空输入框
      newMessage.value = '';
      scrollToBottom();
    });

  };
  
  const scrollToBottom = () => {
    nextTick(() => {
        if (chatMessages.value) {
          chatMessages.value.scrollTop = chatMessages.value.scrollHeight;
        }
      });
  };

  watch(comunicationStore.messagemMap.get(props.diagId), () => {
    console.log('watch');
    const historylist = comunicationStore.messagemMap.get(props.diagId);
    messages.value = [];
    historylist?.forEach((item) => {
      addMessage(item.content, item.senderType);
    })
  });

  const init = async () => {
    // 获取历史消息
    // getHistory(props.diagId);    //这里出错了，暂时注释掉

    let historylist = comunicationStore.messagemMap.get(props.diagId);

    console.log(historylist);

    if (!historylist || historylist.length === 0) {
      await comunicationStore.initMessageMap(doctorId, patientId);
      historylist = comunicationStore.messagemMap.get(props.diagId);
    }

    historylist?.forEach((item) => {
      addMessage(item.content, item.senderType);
    })

    scrollToBottom();
  }

  const markRead = () => {
    DoAxiosWithErro(`/medical/diagnoses/${props.diagId}/feedback/read`, 'post', {}, true).then((res) => {
      comunicationStore.unreadCounters[props.diagId] = 0
    });
  }

  
  onMounted(() => {
    
    // 获取历史消息
    // getHistory(props.diagId);    //这里出错了，暂时注释掉
    init();

    markRead();




    // 连接WebSocket
    // const socket = new SockJS('/ws');
    // const stompClient = new Client({
    //   webSocketFactory() {
    //     return socket;
    //   }
    // });
    // stompClient.connectHeaders = {
    //   "sa-token-authorization":  userStore.userToken
    // };

    // stompClient.onConnect = (frame) => {

    //   stompClient.subscribe('/user/queue/feedback', (response) => {
    //     const data = JSON.parse(response.body); // 假设返回的数据是一个对象
    //     console.log(data);
    //     messages.value.push({
    //       sender: 'doctor',
    //       text: data.message,
    //       avatar: '/src/assets/doctor.png',
    //     });
    //     scrollToBottom();
    //   });
    // };
    // stompClient.activate();
  });
  </script>
  
  <style scoped>
  .chat-container {
    width: 100%;
    max-width: 600px;
    height: 600px;
    margin: 0 auto;
    background-color: #f5f7fa;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
  }
  
  .chat-header {
    background-color: #4a6cf7;
    color: white;
    padding: 15px;
    text-align: center;
  }
  
  .chat-messages {
    flex: 1;
    overflow-y: auto;
    padding: 15px;
    display: flex;
    flex-direction: column;
    gap: 15px;
  }
  
  .message {
    display: flex;
    max-width: 80%;
    margin-bottom: 10px;
  }
  
  .message.patient {
    justify-content: flex-end;
  }
  
  .message.patient .message-avatar {
    order: 2;
  }
  
  .message-avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    overflow: hidden;
    margin: 0 10px;
  }
  
  .message-avatar img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .message-content {
    background-color: white;
    padding: 10px 15px;
    border-radius: 18px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    max-width: 80%;
    word-wrap: break-word;
  }
  
  .message.patient .message-content {
    background-color: #4a6cf7;
    color: white;
    border-radius: 18px 18px 0 18px;
  }
  
  .message-sender {
    font-size: 12px;
    font-weight: bold;
    margin-bottom: 4px;
  }
  
  .message-text {
    font-size: 14px;
  }
  
  .chat-input {
    padding: 15px;
    background-color: white;
    display: flex;
    border-top: 1px solid #e0e0e0;
  }
  
  .textarea-field {
    flex: 1;
    padding: 12px 15px;
    border: 1px solid #e0e0e0;
    border-radius: 20px;
    outline: none;
    font-size: 14px;
    resize: none;
    overflow-y: auto;
    min-height: 60px;
    max-height: 120px;
  }
  
  .send-button {
    background-color: #4a6cf7;
    color: white;
    border: none;
    border-radius: 20px;
    padding: 12px 20px;
    margin-left: 10px;
    cursor: pointer;
    font-size: 14px;
    transition: background-color 0.3s;
  }
  
  .send-button:hover {
    background-color: #3a5bd9;
  }
  </style>