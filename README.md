// MERN Blog Project Structure

// 1. Backend - Node.js + Express
// File: server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/blog', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log('MongoDB connected')).catch(err => console.error('Connection failed', err));

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: String,
  date: { type: Date, default: Date.now }
});
const Post = mongoose.model('Post', postSchema);

// Routes
app.get('/posts', async (req, res) => {
  const posts = await Post.find();
  res.json(posts);
});

app.post('/posts', async (req, res) => {
  const newPost = new Post(req.body);
  await newPost.save();
  res.json(newPost);
});

app.listen(5000, () => console.log('Server running on http://localhost:5000'));


// 2. Frontend - React
// File: src/App.js
import React, { useState, useEffect } from 'react';

function App() {
  const [posts, setPosts] = useState([]);
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  useEffect(() => {
    fetch('http://localhost:5000/posts')
      .then(res => res.json())
      .then(data => setPosts(data));
  }, []);

  const addPost = async () => {
    await fetch('http://localhost:5000/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, content, author: 'Admin' })
    });
    setTitle('');
    setContent('');
  };

  return (
    <div>
      <h2>Blog Posts</h2>
      {posts.map(post => (
        <div key={post._id}>
          <h3>{post.title}</h3>
          <p>{post.content}</p>
        </div>
      ))}
      <h3>Add New Post</h3>
      <input placeholder='Title' value={title} onChange={e => setTitle(e.target.value)} />
      <textarea placeholder='Content' value={content} onChange={e => setContent(e.target.value)} />
      <button onClick={addPost}>Add Post</button>
    </div>
  );
}

export default App;
