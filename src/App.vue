<template>
  <div id="chat-container">
    <div id="chat-messages">
      <div v-for="message in messages" :key="message.id" :class="messageClass(message)">
        {{ message.text }}
      </div>
    </div>
    <div id="user-input">
      <input type="text" placeholder="메시지를 입력하세요..." v-model="userMessage" @keydown.enter="sendMessage" />
      <button @click="sendMessage">전송</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userMessage: '',
      messages: [],  // 대화 목록 저장
      apiEndpoint: 'https://api.openai.com/v1/chat/completions',
      apiKey: process.env.VUE_APP_OPENAI_API_KEY,  // 환경 변수로 토큰 관리
    };
  },
  methods: {
    messageClass(message) {
      return message.isUser ? 'message user-message' : 'message ai-message';
    },
    addUserMessage(message) {
      this.messages.push({ text: message, isUser: true, id: Date.now() });  // push()로 메시지를 아래로 추가
      this.$nextTick(() => this.scrollToBottom());  // 메시지 추가 후 스크롤 맨 아래로 이동
    },
    addAIMessage(message) {
      this.messages.push({ text: message, isUser: false, id: Date.now() });
      this.$nextTick(() => this.scrollToBottom());  // AI 메시지 추가 후 스크롤 맨 아래로 이동
    },
    async fetchAIResponse(prompt) {
      if (prompt.toLowerCase() === "xin chào") {
        return "Xin chào! Tôi có thể giúp gì cho bạn?";
      }
      if (prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ý nghĩa")) {
        return "Điều này có nghĩa là một điều khoản đặc biệt được đồng ý và điền vào các tài liệu chính thức sẽ được lập.";
      }
      if (prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("lý do")) {
        return "Việc ghi rõ các điều khoản đặc biệt có thể ngăn ngừa hoặc giải quyết các tranh chấp có thể phát sinh sau này, và vì hợp đồng bằng miệng không có hiệu lực pháp lý, nên tốt nhất là cần được lập thành văn bản.";
      }
      if (prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("hiệu lực")) {
        return "'Điều khoản đặc biệt' là một cam kết đặc biệt giữa các bên. Hợp đồng mua bán bất động sản là một 'hợp đồng tư nhân' được ký kết giữa cá nhân với cá nhân. Do đó, dù có 규 định khác trong luật pháp, các điều khoản đặc biệt giữa các bên vẫn được ưu tiên áp dụng.\n" +
            "\n" +
            "Tuy nhiên, dù là điều khoản đặc biệt, nếu vi phạm các 규정 bắt buộc gây hại cho trật tự xã hội (các 규정 pháp luật liên quan đến trật tự công cộng), thì sẽ không có.\n" +
            "\n" +
            "Ngoài ra, trong trường hợp hành vi của chủ nhà liên quan đến việc hoàn trả tiền đặt cọc bị coi là vi phạm các điều khoản đặc biệt trong hợp đồng, việc có được bồi thường hay không cũng có thể khác nhau tùy theo luật pháp liên quan. Do đó, để xác định liệu bạn có thể nhận được bồi thường hợp pháp hay không, tốt nhất là tham khảo ý kiến của chuyên gia pháp lý hoặc luật sư tại quốc gia đó.";
      }
      if (prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("lưu ý")) {
        return "Có một số trường hợp hiểu lầm rằng chỉ cần ghi chú ý vào điều khoản đặc biệt thì sẽ có pháp lý, nhưng điều này không đúng. \n" +
            "\n" +
            "Các điều khoản đặc biệt trái với quy định bắt buộc sẽ không có pháp lý, ngay cả khi được lập thành văn bản.\n" +
            "\n" +
            "Nói cách khác, các điều khoản đặc biệt vi phạm pháp luật sẽ không có pháp lý ngay cả khi được ghi vào hợp đồng.\n" +
            "\n" +
            "Do đó, ngay cả khi ghi các nội dung vi phạm pháp luật để ép buộc hoặc ràng buộc đối phương, điều này cũng không có pháp lý. Nếu bạn ghi các nội dung vi phạm pháp luật và ép buộc đối phương chấp nhận, bạn có thể sẽ tự gây thiệt hại cho chính mình.";
      }
      if ((prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("vay tiền thuê nhà")) || (prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu") && prompt.toLowerCase().includes("vay tiền thuê nhà"))) {
        return "Có một số câu văn đề xuất cho điều khoản đặc biệt về khoản vay tiền đặt cọc nhà thuê.\n" +
            "\n" +
            "- “Trong trường hợp không thể vay tiền đặt cọc do khiếm khuyết của tài sản, hợp đồng sẽ bị hủy bỏ và tiền đặt cọc sẽ được hoàn trả.”\n" +
            "- Người cho thuê sẽ tích cực hợp tác trong việc vay tiền đặt cọc nhà thuê.\n" +
            "- Trong trường hợp không thể tham gia bảo hiểm hoàn trả tiền đặt cọc, hợp đồng sẽ bị hủy bỏ và tiền đặt cọc sẽ được hoàn trả.\n" +
            "(Trước khi lập hợp đồng, nên đến ngân hàng để xác nhận khả năng vay và hạn mức trước khi tiến hành.)\n" +
            "- Tuy nhiên, nếu tiền vay không được giải ngân do vấn đề của người thuê, người thuê sẽ không được hoàn lại tiền đặt cọc hoặc tiền bảo đảm.\n" +
            "\n" +
            "Tôi khuyên bạn nên tham khảo ý kiến của chuyên gia để có được thông tin chính xác hoặc chi tiết hơn.";
      }
      if ((prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("quyền lợi của người thuê nhà ghi trên hợp đồng ")) || (prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu") && prompt.toLowerCase().includes("quyền lợi của người thuê nhà ghi trên hợp đồng "))) {
        return "Có một số câu văn đề xuất cho điều khoản đặc biệt về quyền yêu cầu bảo vệ quyền lợi.\n" +
            "\n" +
            "- “Không thiết lập quyền thế chấp hoặc các quyền lợi khác cho đến ngày tiếp theo sau ngày thanh toán số dư.”\n" +
            "- “Quyền thế chấp có ưu tiên sẽ bị xóa trước ngày thanh toán số dư.”\n" +
            "- Vì khả năng phản kháng có từ 0h ngày tiếp theo sau khi đăng ký chuyển đổi địa chỉ cư trú, điều khoản đặc biệt phải quy định không được thiết lập quyền lợi trước đó.\n" +
            "- Quyền thế chấp có ngay trong ngày thiết lập, nhưng ngày đăng ký chuyển đổi địa chỉ cư trú và ngày xác nhận chỉ có từ ngày hôm sau vì nhận được sau khi đăng ký.\n" +
            "\n" +
            "Tôi khuyên bạn nên tham khảo ý kiến của chuyên gia để có được thông tin chính xác hoặc chi tiết hơn.";
      }
      if ((prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("thuế nhà nước")) || (prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu") && prompt.toLowerCase().includes("thuế nhà nước"))) {
        return "Có một số câu văn đề xuất cho điều khoản đặc biệt về việc xác nhận hoàn thành thuế quốc gia.\n" +
            "\n" +
            "- “Nhằm bảo vệ người thuê nhà trước việc không nộp thuế.”\n" +
            "- Trong trường hợp người cho thuê chưa nộp thuế quốc gia hoặc thuế địa phương, hợp đồng sẽ bị hủy bỏ.\n" +
            "\n" +
            "Tôi khuyên bạn nên tham khảo ý kiến của chuyên gia để có được thông tin chính xác hoặc chi tiết hơn.";
      }
      if ((prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("quyền được hoàn trả tiền thuê nhà")) || (prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu") && prompt.toLowerCase().includes("quyền được hoàn trả tiền thuê nhà"))) {
        return "Có một số câu văn đề xuất cho điều khoản đặc biệt về bảo đảm hoàn trả tiền đặt cọc.\n" +
            "- “Trong trường hợp không thể tham gia bảo hiểm đảm bảo hoàn trả tiền đặt cọc, hợp đồng sẽ bị hủy bỏ.”\n" +
            "Điều quan trọng là phải kiểm tra trước và xác nhận xem bảo hiểm đảm bảo hoàn trả tiền đặt cọc có khả thi hay không trước khi ký hợp đồng.\n" +
            "Tôi khuyên bạn nên tham khảo ý kiến của chuyên gia để có thông tin chính xác và chi tiết hơn.";
      }
      if ((prompt.toLowerCase().includes("điều khoản đặc biệt") && prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu")) || (prompt.toLowerCase().includes("ví dụ") && prompt.toLowerCase().includes("câu") && prompt.toLowerCase().includes("nội thất"))) {
        return "Có một số điều khoản đặc biệt liên quan đến các lỗi về nội thất, vật dụng, giấy dán tường và sàn nhà đã được thỏa thuận trước khi dọn vào nhà như sau. \n" +
            "- “Trong trường hợp hư hỏng bên trong do sự bất cẩn của người thuê, người thuê phải khôi phục lại như ban đầu, các hư hỏng lớn do người cho thuê chịu trách nhiệm, và các hư hỏng nhỏ (chi phí sửa chữa dưới 5 triệu đồng, các vật phẩm tiêu hao) do người thuê chịu trách nhiệm.”\n" +
            "“Việc dán giấy dán tường sẽ do người cho thuê chịu trách nhiệm và hoàn thành trước khi dọn vào ở.”\n" +
            "Để có thông tin chính xác hơn hoặc chi tiết hơn, bạn nên tư vấn với chuyên gia.";
      }

      const requestOptions = {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.apiKey}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [
            {
              "role": "system",
              "content": "너는 한국 부동산에 관해서 전문가인데, 베트남인을 상대로 한국의 부동산 문화에 대해서 설명을 해 줘야하는 상황이야. 베트남어로 친절하게 대답을 해줘."
            },
            {
              role: "user",
              content: prompt
            }
          ],
          temperature: 0.8,
          max_tokens: 100,
          top_p: 1,
          frequency_penalty: 0.5,
          presence_penalty: 0.5,
        }),
      };

      try {
        const response = await fetch(this.apiEndpoint, requestOptions);
        const data = await response.json();
        return data.choices[0].message.content;
      } catch (error) {
        console.error('OpenAI API 호출 중 오류 발생:', error);
        return 'OpenAI API 호출 중 오류 발생';
      }
    },
    async sendMessage() {
      if (this.userMessage.trim() === '') return;
      this.addUserMessage(this.userMessage);
      const aiResponse = await this.fetchAIResponse(this.userMessage);
      this.addAIMessage(aiResponse);
      this.userMessage = '';
    },
    scrollToBottom() {
      const chatMessages = document.getElementById('chat-messages');
      chatMessages.scrollTop = chatMessages.scrollHeight;  // 채팅 창을 맨 아래로 스크롤
    },
  },
};
</script>

<style scoped>
body {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  background-color: #f0f0f0;
}

#chat-container {
  width: 400px;
  height: 600px;
  display: flex;
  flex-direction: column;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background-color: #ffffff;
}

#chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
  display: flex;
  flex-direction: column;
}

.message {
  border-radius: 15px;
  padding: 10px;
  margin-bottom: 10px;
  max-width: 70%;
  word-break: break-word;
}

.user-message {
  align-self: flex-end;
  background-color: #1e88e5;
  color: #ffffff;
}

.ai-message {
  align-self: flex-start;
  background-color: #e6e6e6;
}

#user-input {
  display: flex;
  padding: 10px;
  border-top: 1px solid #ddd;
  background-color: #ffffff;
}

#user-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 15px;
  outline: none;
}

#user-input button {
  border: none;
  background-color: #1e88e5;
  color: white;
  padding: 10px 15px;
  margin-left: 10px;
  border-radius: 15px;
  cursor: pointer;
}
</style>