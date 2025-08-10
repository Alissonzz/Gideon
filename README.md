    <!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Chat IA Gideon com Voz</title>
  <style>
    body { font-family: Arial, sans-serif; background: #121212; color: #0f0; }
    #chat { max-width: 600px; margin: 20px auto; }
    #messages { height: 300px; overflow-y: scroll; border: 1px solid #0f0; padding: 10px; background: #000; }
    input, button { padding: 10px; font-size: 16px; margin-top: 10px; }
    input { width: 70%; background: #222; color: #0f0; border: 1px solid #0f0; }
    button { background: #0f0; color: #000; border: none; cursor: pointer; margin-left: 5px; padding: 10px 15px; }
  </style>
</head>
<body>
  <div id="chat">
    <div id="messages"></div>
    <input type="text" id="input" placeholder="Digite sua pergunta ou fale..." />
    <button id="sendBtn">Enviar</button>
    <button id="voiceBtn">🎤 Falar</button>
  </div>

  <script>
    const messages = document.getElementById('messages');
    const input = document.getElementById('input');
    const sendBtn = document.getElementById('sendBtn');
    const voiceBtn = document.getElementById('voiceBtn');

    sendBtn.onclick = () => {
      sendMessage(input.value);
      input.value = '';
    };

    // Envia mensagem para backend e recebe resposta
    async function sendMessage(text) {
      if (!text.trim()) return;
      appendMessage('Você', text);

      try {
        const res = await fetch('http://localhost:3000/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message: text }),
        });
        const data = await res.json();
        appendMessage('Gideon', data.reply);
        speakText(data.reply);
      } catch (e) {
        appendMessage('Erro', 'Não foi possível conectar ao servidor.');
      }
    }

    // Adiciona mensagem na tela
    function appendMessage(sender, text) {
      messages.innerHTML += `<p><strong>${sender}:</strong> ${text}</p>`;
      messages.scrollTop = messages.scrollHeight;
    }

    // Fala o texto usando Speech Synthesis
    function speakText(text) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = 'pt-BR';
      speechSynthesis.speak(utterance);
    }

    // Reconhecimento de voz
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    const recognition = SpeechRecognition ? new SpeechRecognition() : null;
    if (recognition) {
      recognition.lang = 'pt-BR';
      recognition.interimResults = false;

      recognition.onresult = event => {
        const speechToText = event.results[0][0].transcript;
        input.value = speechToText;
        sendMessage(speechToText);
      };

      recognition.onerror = event => {
        appendMessage('Erro', 'Reconhecimento de voz falhou: ' + event.error);
      };

      voiceBtn.onclick = () => {
        recognition.start();
      };
    } else {
      voiceBtn.disabled = true;
      voiceBtn.textContent = 'Microfone não suportado';
    }
  </script>
</body>
</html>
            
