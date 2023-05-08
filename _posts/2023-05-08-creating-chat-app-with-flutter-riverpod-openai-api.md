---
layout: post
title: "Creating a Chat App with Flutter, Riverpod, and OpenAI API (ChatGPT)"
date: 2023-05-08
author: Tal Nir
categories: [Flutter, Riverpod, OpenAI, ChatGPT]
---

## In this tutorial, we will create a chat app using Flutter, Riverpod, and OpenAI API. We will use the OpenAI API to generate responses based on user input, and Riverpod for state management. Here is the outline of the steps:

1.  **Update `pubspec.yaml`**: Add dependencies for Flutter Hooks, Hooks Riverpod, and HTTP. These libraries are necessary for managing the state of the application, handling user input, and making HTTP requests to the OpenAI API.
   
```yaml
name: chat_ai  
description: A new Flutter project.  

# Prevent accidental publishing to pub.dev.  
publish_to: 'none'  
  
version: 1.0.0+1  
  
environment:  
  sdk: '>=2.19.6 <3.0.0'  
  
dependencies:  
  flutter:  
    sdk: flutter  
  flutter_localizations:  
    sdk: flutter  
  
  flutter_hooks: ^0.18.0  
  hooks_riverpod: ^2.3.6  
  http: ^0.13.6  
  
dev_dependencies:  
  flutter_test:  
    sdk: flutter  
  
  flutter_lints: ^2.0.0  
  
flutter:  
  uses-material-design: true  
  
  # Enable generation of localized Strings from arb files.  
  generate: true  
  
  assets:  
    - assets/images/
```


2.  **Create `main.dart`**: Set up the MaterialApp and run the application. This file initializes the app, sets the theme, and specifies the home screen (in this case, ChatScreen).
```dart
import 'package:chat_ai/src/view/chat_screen.dart';  
import 'package:flutter/material.dart';  
import 'package:hooks_riverpod/hooks_riverpod.dart';  
  
void main() {  
  runApp(const ProviderScope(child: MyApp()));  
}  
  
class MyApp extends StatelessWidget {  
  const MyApp({super.key});  
  
  @override  
  Widget build(BuildContext context) {  
    return MaterialApp(  
      title: 'Chat App',  
  theme: ThemeData(primarySwatch: Colors.blue),  
  home: const ChatScreen(),  
  );  
  }  
}
``` 

3.  **Create `api_service.dart`**: Handle API requests to the OpenAI API. This file defines the ApiService class responsible for making HTTP requests to the OpenAI API. The `fetchResponse` method sends a user message to the API and retrieves a generated response.
```dart
import 'dart:convert';  
import 'package:http/http.dart' as http;  
  
class ApiService {  
  final String apiKey;  
  
  ApiService(this.apiKey);  
  
  Future<String> fetchResponse(String message) async {  
    const url = 'https://api.openai.com/v1/engines/davinci-codex/completions';  
 final headers = {  
      'Content-Type': 'application/json',  
  'Authorization': 'Bearer $apiKey'  
  };  
  
 final body = json.encode({  
      'prompt': 'ChatGPT: $message',  
  'max_tokens': 50,  
  'n': 1,  
  'stop': ['ChatGPT:'],  
  'temperature': 0.5,  
  });  
  
 final response = await http.post(Uri.parse(url), headers: headers, body: body);  
  
 if (response.statusCode == 200) {  
      final data = json.decode(response.body);  
 return data['choices'][0]['text'].trim();  
  } else {  
      throw Exception('Failed to fetch response from OpenAI API: ${response.body}');  
  }  
  }  
}
```

4.  **Create `message_provider.dart`**: Define the API service provider. This file uses Riverpod to create a provider for the ApiService. The provider allows us to easily access the ApiService instance throughout the app.

```dart
import 'package:hooks_riverpod/hooks_riverpod.dart';  
  
import '../repository/api_service.dart';  
  
const apiKey = 'your_openai_api_key';  
final apiServiceProvider = Provider<ApiService>((ref) => ApiService(apiKey));
```

5.  **Create `chat_screen.dart`**: Build the main screen of the chat app.
    
    -   Set up the UI components: Create a TextField for user input and a ListView to display messages. The ListView.builder is used to generate message bubbles for both the user and the AI.
    -   Implement the sendMessage function: This function is triggered when the user submits a message. It sends the message to the OpenAI API using the `apiService.fetchResponse` method and appends the response to the messages list. The UI is updated accordingly to show both the user's message and the AI's response.
```dart
import 'package:flutter/material.dart';  
import 'package:flutter_hooks/flutter_hooks.dart';  
import 'package:hooks_riverpod/hooks_riverpod.dart';  
  
import '../model/message_provider.dart';  
  
class ChatScreen extends HookConsumerWidget {  
  const ChatScreen({super.key});  
  
  @override  
  Widget build(BuildContext context, WidgetRef ref) {  
    final TextEditingController controller = useTextEditingController();  
 final scrollController = ScrollController();  
 final messages = useState<List<Map<String, String>>>([]);  
 final apiService = ref.watch(apiServiceProvider);  
  
 void sendMessage() async {  
      if (controller.text.isNotEmpty) {  
        final message = controller.text;  
  messages.value = List.from(messages.value)  
          ..add({"type": "user", "text": message});  
  controller.clear();  
  
 try {  
          final response = await apiService.fetchResponse(message);  
  messages.value = List.from(messages.value)  
            ..add({"type": "chatgpt", "text": response});  
  scrollController.animateTo(  
            scrollController.position.maxScrollExtent,  
  duration: const Duration(milliseconds: 500),  
  curve: Curves.easeInOut,  
  );  
  } catch (e) {  
          print('Error fetching response: $e');  
  }  
      }  
    }  
  
    return Scaffold(  
      appBar: AppBar(title: const Text('Chat App')),  
  body: Container(  
        decoration: const BoxDecoration(  
          image: DecorationImage(  
            image: AssetImage("assets/images/background.jpg"),  
  fit: BoxFit.cover,  
  ),  
  ),  
  child: Column(  
          children: [  
            Expanded(  
              child: ListView.builder(  
                controller: scrollController,  
  itemCount: messages.value.length,  
  itemBuilder: (context, index) {  
                  final message = messages.value[index];  
 final isUser = message["type"] == "user";  
 final align = isUser  
                      ? CrossAxisAlignment.end  
  : CrossAxisAlignment.start;  
 final color =  
                      isUser ? Colors.greenAccent[700] : Colors.grey.shade300;  
 final txtColor = isUser ? Colors.white : Colors.black;  
  
 return Padding(  
                    padding: const EdgeInsets.symmetric(  
                        horizontal: 16.0, vertical: 4.0),  
  child: Column(  
                      crossAxisAlignment: align,  
  children: [  
                        Container(  
                          constraints: BoxConstraints(  
                            maxWidth: MediaQuery.of(context).size.width * 0.75,  
  ),  
  padding: const EdgeInsets.symmetric(  
                              horizontal: 16.0, vertical: 10.0),  
  decoration: BoxDecoration(  
                            color: color,  
  borderRadius: BorderRadius.circular(20.0),  
  ),  
  child: Text(  
                            message["text"] ?? "",  
  style: TextStyle(color: txtColor, fontSize: 16),  
  ),  
  ),  
  ],  
  ),  
  );  
  },  
  ),  
  ),  
  Padding(  
              padding:  
                  const EdgeInsets.symmetric(horizontal: 8.0, vertical: 4.0),  
  child: Container(  
                decoration: BoxDecoration(  
                  color: Colors.white,  
  borderRadius: BorderRadius.circular(25.0),  
  ),  
  child: Row(  
                  children: [  
                    Expanded(  
                      child: Padding(  
                        padding: const EdgeInsets.symmetric(horizontal: 8.0),  
  child: TextField(  
                          controller: controller,  
  decoration: const InputDecoration.collapsed(  
                              hintText: 'Type your message'),  
  onSubmitted: (value) => sendMessage(),  
  ),  
  ),  
  ),  
  IconButton(  
                      onPressed: sendMessage,  
  icon: Icon(Icons.send, color: Colors.greenAccent[700]),  
  ),  
  ],  
  ),  
  ),  
  ),  
  ],  
  ),  
  ),  
  );  
  }  
}
```

6.  **background image `assets/images/background.png`**
 To find a suitable background image for your chat app, you can search for royalty-free images on websites like Unsplash, Pexels, or Pixabay. These websites offer a wide variety of high-quality images that can be used for free.

For example, you can use this image from Unsplash as a chat background:

[https://unsplash.com/photos/Oaqk7qqNh_c](https://unsplash.com/photos/Oaqk7qqNh_c)

To use this image in your app:

1.  Download the image and save it in your project's `assets/images` folder. If the folder does not exist, create it.
    
2.  If you haven't done so already, create the `background.jpg` file in the `assets/images` folder and paste the downloaded image there. Alternatively, you can save the image with a different name and update the AssetImage path in the `ChatScreen` code accordingly.

Now, the background image should be applied to your chat screen, giving it a more appealing look.

Remember that you should always respect the licensing terms of any image you use. The websites I mentioned typically allow the use of their images for free, even for commercial purposes, but it's always a good idea to double-check the specific image's licensing terms before using it in your project.