import 'dart:convert';
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:flutter_tts/flutter_tts.dart';
import 'package:speech_to_text/speech_to_text.dart' as stt;
import 'package:image_picker/image_picker.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(const KioApp());
}

class KioApp extends StatelessWidget {
  const KioApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'KIO',
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark().copyWith(
        scaffoldBackgroundColor: const Color(0xFF030712),
        colorScheme: const ColorScheme.dark(
          primary: Color(0xFF00F0FF),
          secondary: Color(0xFF7000FF),
        ),
      ),
      home: const KioHomeScreen(),
    );
  }
}

class KioHomeScreen extends StatefulWidget {
  const KioHomeScreen({super.key});

  @override
  State<KioHomeScreen> createState() => _KioHomeScreenState();
}

class _KioHomeScreenState extends State<KioHomeScreen> with TickerProviderStateMixin {
  final TextEditingController _apiKeyController = TextEditingController();
  final TextEditingController _textController = TextEditingController();
  
  late stt.SpeechToText _speech;
  late FlutterTts _tts;
  late AnimationController _animController;

  bool _isListening = false;
  bool _isProcessing = false;
  String _statusText = "Awaiting command, Ashaz...";
  File? _selectedImage;
  final ImagePicker _picker = ImagePicker();

  @override
  void initState() {
    super.initState();
    _speech = stt.SpeechToText();
    _tts = FlutterTts();
    _setupTts();

    _animController = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 3),
    )..repeat(reverse: true);
  }

  void _setupTts() async {
    await _tts.setLanguage("en-US");
    await _tts.setPitch(0.9); // Techy Jarvis feel
    await _tts.setSpeechRate(1.0);
  }

  Future<void> _listen() async {
    if (!_isListening) {
      bool available = await _speech.initialize();
      if (available) {
        setState(() => _isListening = true);
        _speech.listen(
          onResult: (val) {
            if (val.finalResult) {
              setState(() => _isListening = false);
              _processInput(val.recognizedWords);
            }
          },
        );
      }
    } else {
      setState(() => _isListening = false);
      _speech.stop();
    }
  }

  Future<void> _pickImage() async {
    final XFile? image = await _picker.pickImage(source: ImageSource.gallery);
    if (image != null) {
      setState(() {
        _selectedImage = File(image.path);
        _statusText = "Photo selected! Ask KIO about this outfit/tech.";
      });
    }
  }

  Future<void> _processInput(String query) async {
    if (query.trim().isEmpty) return;
    
    final apiKey = _apiKeyController.text.trim();
    if (apiKey.isEmpty) {
      _speak("Ashaz bro, pehle settings me Groq API key toh daal de!");
      return;
    }

    setState(() {
      _isProcessing = true;
      _statusText = "KIO Analyzing...";
    });

    try {
      String responseText = "";
      if (_selectedImage != null) {
        // Image + Text Processing via Groq Vision Model
        responseText = await _analyzeImageWithGroq(_selectedImage!, query, apiKey);
      } else {
        // Text/Voice Processing via Llama-3.3
        responseText = await _chatWithGroq(query, apiKey);
      }

      setState(() {
        _statusText = responseText;
        _selectedImage = null; // Reset image after query
      });

      _speak(responseText);
    } catch (e) {
      setState(() => _statusText = "System Connection Error!");
    } finally {
      setState(() => _isProcessing = false);
    }
  }

  Future<String> _chatWithGroq(String prompt, String apiKey) async {
    final response = await http.post(
      Uri.parse('https://api.groq.com/openai/v1/chat/completions'),
      headers: {
        'Authorization': 'Bearer $apiKey',
        'Content-Type': 'application/json',
      },
      body: jsonEncode({
        "model": "llama-3.3-70b-versatile",
        "messages": [
          {
            "role": "system",
            "content": "You are KIO, Ashaz's futuristic Jarvis-like AI buddy. You are a tech wizard AND a sharp fashion/streetwear expert. Speak casually in Hinglish (mix of English and Hindi). Keep answers brief (under 3 sentences), funny, witty, and deeply helpful."
          },
          {"role": "user", "content": prompt}
        ]
      }),
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return data['choices'][0]['message']['content'];
    }
    return "API Error connection fail ho gaya bhai.";
  }

  Future<String> _analyzeImageWithGroq(File imageFile, String prompt, String apiKey) async {
    List<int> imageBytes = await imageFile.readAsBytes();
    String base64Image = base64Encode(imageBytes);

    final response = await http.post(
      Uri.parse('https://api.groq.com/openai/v1/chat/completions'),
      headers: {
        'Authorization': 'Bearer $apiKey',
        'Content-Type': 'application/json',
      },
      body: jsonEncode({
        "model": "llama-3.2-11b-vision-preview",
        "messages": [
          {
            "role": "user",
            "content": [
              {
                "type": "text",
                "text": "You are KIO, Ashaz's personal style and tech guru. Analyze this photo. Prompt: $prompt. Keep advice short, trendy, and stylish in Hinglish."
              },
              {
                "type": "image_url",
                "image_url": {"url": "data:image/jpeg;base64,$base64Image"}
              }
            ]
          }
        ]
      }),
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return data['choices'][0]['message']['content'];
    }
    return "Image read nahi ho paayi bro!";
  }

  void _speak(String text) async {
    await _tts.speak(text);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("K I O", style: TextStyle(letterSpacing: 4, fontWeight: FontWeight.bold, color: Color(0xFF00F0FF))),
        backgroundColor: Colors.transparent,
        elevation: 0,
        actions: [
          IconButton(
            icon: const Icon(Icons.key, color: Color(0xFF00F0FF)),
            onPressed: _showApiKeyDialog,
          )
        ],
      ),
      body: SafeArea(
        child: Column(
          children: [
            const SizedBox(height: 20),
            // Futuristic Hologram Orb Visualizer
            Center(
              child: AnimatedBuilder(
                animation: _animController,
                builder: (context, child) {
                  return Container(
                    width: 180,
                    height: 180,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      gradient: RadialGradient(
                        colors: _isListening
                            ? [const Color(0xFFFF0055), const Color(0xFF7000FF), Colors.transparent]
                            : [const Color(0xFF00F0FF), const Color(0xFF0040FF), Colors.transparent],
                      ),
                      boxShadow: [
                        BoxShadow(
                          color: _isListening ? const Color(0xFFFF0055) : const Color(0xFF00F0FF),
                          blurRadius: 30 + (_animController.value * 20),
                          spreadRadius: 2,
                        )
                      ],
                    ),
                    child: Center(
                      child: Container(
                        width: 140,
                        height: 140,
                        decoration: BoxDecoration(
                          shape: BoxShape.circle,
                          border: Border.all(color: const Color(0xFF00F0FF), width: 1.5),
                        ),
                        child: const Icon(Icons.blur_on, size: 70, color: Colors.white),
                      ),
                    ),
                  );
                },
              ),
            ),
            
            const SizedBox(height: 20),
            
            // Image Preview (if selected)
            if (_selectedImage != null)
              Container(
                height: 70,
                width: 70,
                decoration: BoxDecoration(
                  borderRadius: BorderRadius.circular(10),
                  border: Border.all(color: const Color(0xFF00F0FF)),
                  image: DecorationImage(image: FileImage(_selectedImage!), fit: BoxFit.cover),
                ),
              ),

            // Status / Output Screen
            Expanded(
              child: Container(
                padding: const EdgeInsets.all(20),
                alignment: Alignment.center,
                child: SingleChildScrollView(
                  child: Text(
                    _statusText,
                    textAlign: TextAlign.center,
                    style: const TextStyle(fontSize: 16, color: Colors.white, height: 1.4),
                  ),
                ),
              ),
            ),

            // Controls (Voice, Image Upload, Silent Typing)
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 15, vertical: 10),
              decoration: const BoxDecoration(
                color: Color(0xFF0B1329),
                borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
              ),
              child: Row(
                children: [
                  IconButton(
                    icon: const Icon(Icons.add_a_photo, color: Color(0xFF00F0FF)),
                    onPressed: _pickImage,
                  ),
                  Expanded(
                    child: TextField(
                      controller: _textController,
                      decoration: const InputDecoration(
                        hintText: "Silent mode? Type here...",
                        hintStyle: TextStyle(color: Colors.grey, fontSize: 13),
                        border: InputBorder.none,
                      ),
                    ),
                  ),
                  IconButton(
                    icon: const Icon(Icons.send, color: Color(0xFF00F0FF)),
                    onPressed: () {
                      _processInput(_textController.text);
                      _textController.clear();
                    },
                  ),
                  IconButton(
                    icon: Icon(_isListening ? Icons.mic : Icons.mic_none, color: _isListening ? Colors.red : const Color(0xFF00F0FF)),
                    onPressed: _listen,
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }

  void _showApiKeyDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        backgroundColor: const Color(0xFF0B1329),
        title: const Text("Groq API Key"),
        content: TextField(
          controller: _apiKeyController,
          decoration: const InputDecoration(hintText: "Paste key here"),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text("Save"),
          )
        ],
      ),
    );
  }
}
