
Query selfhosted AI  
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>KIZIDS</title>
  <style>
    body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 0;
}

h1 {
  color: #333;
  text-align: center;
  padding: 20px 0;
  background-color: #fff;
  margin-bottom: 30px;
}

.chat-entry {
  width: 100%;
  margin-bottom: 20px;
}

.timestamp {
  font-weight: bold;
  color: #555;
}

.role {
  font-weight: bold;
}

.content {
  display: inline-block;
  max-width: 80%;
  word-wrap: break-word;
}

.user-message {
  background-color: #fff;
  color: #000;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 15px;
}

.assistant-message {
  background-color: #fff;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

input,
select {
  width: 100%;
  padding: 8px;
  margin-bottom: 15px;
  box-sizing: border-box;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #3498db;
  color: #fff;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #2980b9;
}

#loadingSpinner {
  display: none;
  margin-top: 10px;
}

.loading {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-radius: 50%;
  border-top: 4px solid #3498db;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
  margin-right: 10px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

#chatForm {
  max-width: 800px;
  margin: 0 auto;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

#responseContainer {
  max-width: 800px;
  margin: 0 auto;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}



#executionTime {
  margin-top: 10px;
  color: #555;
}

#previousresponse {
  margin-top: 20px;
  padding-top: 10px;
  border-top: 1px solid #ccc;
}

#chatHistoryContainer {
   max-width: 800px;
  margin: 0 auto;
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

  </style>
</head>
<body>

  <h1>KIZIDS</h1>

  <form id="chatForm">
    <label for="userMessage">Deine Nachricht:</label>
    <input type="text" id="userMessage" name="userMessage" required>

    <button type="button" onclick="submitChat()">Submit</button>
    <div id="loadingSpinner" class="loading"></div>
  </form>

  <div id="responseContainer">
    <h4>Antwort des Assistenten</h4>
    <div id="executionTime"></div>
    <div id="response"></div>
    <div id="previousresponse"></div>
  </div>

  <div id="chatHistoryContainer"></div>

  <script>
    let serverhistory = [{"role": "assistant", "content": "Sei ein freundlicher Assistent. Antworte kurz und präzise. Antworte immer auf deutsch. Benutze immer die Du-Form. Duze mich."} ];
    tokens = 0;
    function submitChat() {
      tokens = 0;
      var startTime = new Date().getTime();
      var loadingSpinner = document.getElementById('loadingSpinner');
      loadingSpinner.style.display = 'inline-block';

      var userMessage = document.getElementById('userMessage').value;

      serverhistory.push ( { "role": "user", "content": userMessage } );

      var streamOption = "true";

      var xhr = new XMLHttpRequest();
      xhr.open('POST', 'https://xxx.de:4891/v1/chat/completions', true);
      xhr.setRequestHeader('Content-Type', 'application/json');
      console.log(serverhistory);
      var requestBody = {
        "prompt": userMessage,
        "mode": "instruct",
        "temperature": 0.2,
        "max_tokens": 4096,
        "stream": streamOption === "true",
        "messages": serverhistory
      };

      xhr.onreadystatechange = function () {
        var responseDIV = document.getElementById('response');
        var previousresponseDIV = document.getElementById('previousresponse');
        if (xhr.readyState == 3) {
          handleStreamingData(xhr.responseText, startTime, userMessage);
          previousresponseDIV.innerHTML = "";
        }

        if (xhr.readyState == 4) {
          loadingSpinner.style.display = 'none';

          serverhistory.push({"role": "assistant", "content": extractedText})
          displayChatHistory();

          if (xhr.status == 200) {
            // The final response, if any, will be handled in the handleStreamingData function
          } else {
            console.error('Fehler beim Abrufen der Antwort. Statuscode: ' + xhr.status);
          }
        }
      };

      xhr.send(JSON.stringify(requestBody));
    }

    extractedText = ""
    
    function handleStreamingData(chunk, startTime, userMessage) {
      var executionTimeDiv = document.getElementById('executionTime');
      var responseDiv = document.getElementById('response');
      var streamOption = "true";

      if (streamOption === "true") {
        const dataBlocks = chunk.split('data: ');

        let aufgebauterText = "";
        executionTimeDiv.innerHTML = "";
        responseDiv.innerHTML = "";

        for (let i = 1; i < dataBlocks.length; i++) {
          tokens++;
          try {
            const data = JSON.parse(dataBlocks[i]);
            const contentText = data.choices[0].delta.content;
            aufgebauterText += contentText + "";

            extractedText = aufgebauterText.split('###')[0].trim();
            responseDiv.innerHTML = extractedText + "<br><hr><br>";
          } catch (error) {
            console.error('Fehler beim Parsen von JSON-Daten:', error);
          }
        }


        var executionTime = new Date().getTime() - startTime;
        executionTimeDiv.innerHTML = '<strong>Zeit:</strong> ' + (executionTime / 1000).toFixed(1) + ' Sekunden ('+tokens+ " Tokens)<p>";

      }
    }

    function displayChatHistory() {
      let chatHistoryContainer = document.getElementById('chatHistoryContainer');
      chatHistoryContainer.innerHTML = ''; // Leere das Container-Element

      for (let i = serverhistory.length - 1; i > 0; i--) {
        const entry = serverhistory[i];
        chatHistoryContainer.innerHTML += getChatEntryHTML(entry);
      }
    }
    
    function getChatEntryHTML(entry) {
      let roleClass = entry.role === 'user' ? 'user-message' : 'assistant-message';

      return `
        <div class="chat-entry ${roleClass}">
          <span class="timestamp">${new Date().toLocaleString()}</span>
          <span class="role">${entry.role}:</span>
          <span class="content">${entry.content}</span>
        </div>
      `;
    }
    

    document.addEventListener('DOMContentLoaded', function() {
      // Attach an event listener to the input field for user messages
      document.getElementById('userMessage').addEventListener('keypress', function(event) {
        if (event.key === 'Enter') {
          // If the pressed key is 'Enter', prevent the default action (form submission) and call submitChat
          event.preventDefault();
          submitChat();
        }
      });
    });
  </script>

</body>
</html>
```
