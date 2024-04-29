import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() {
  runApp(MyApp());
}

class News {
  final String title;
  final String description;
  final String imageUrl;

  News({required this.title, required this.description, required this.imageUrl});
}

Future<List<News>> fetchNews() async {
  final response = await http.get(Uri.parse('https://your-news-api.com'));
  if (response.statusCode == 200) {
    final List<dynamic> data = json.decode(response.body);
    return data.map((json) => News(
      title: json['title'],
      description: json['description'],
      imageUrl: json['imageUrl'],
    )).toList();
  } else {
    throw Exception('Failed to load news');
  }
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'News Feed',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text('News Feed'),
        ),
        body: NewsFeed(),
      ),
    );
  }
}

class NewsFeed extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<News>>(
      future: fetchNews(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return Center(child: CircularProgressIndicator());
        } else if (snapshot.hasError) {
          return Center(child: Text('Error: ${snapshot.error}'));
        } else {
          final newsList = snapshot.data!;
          return ListView.builder(
            itemCount: newsList.length,
            itemBuilder: (context, index) {
              final news = newsList[index];
              return ListTile(
                title: Text(news.title),
                subtitle: Text(news.description),
                leading: Image.network(news.imageUrl),
              );
            },
          );
        }
      },
    );
  }
}
