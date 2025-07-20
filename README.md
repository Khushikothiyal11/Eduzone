import React from "react";
import { BrowserRouter as Router, Routes, Route, Navigate } from "react-router-dom";

// Pages
import Feed from "./pages/Feed";
import CreatePost from "./pages/CreatePost";
import Login from "./pages/Login";
import Register from "./pages/Register";

// Components
import Navbar from "./components/Navbar";

// ----- Auth Helper -----
const ProtectedRoute = ({ children }) => {
  const token = localStorage.getItem("token");
  if (!token) {
    // Not logged in → redirect to login
    return <Navigate to="/login" replace />;
  }
  return children;
};

function App() {
  return (
    <Router>
      {/* Global Top Navbar */}
      <Navbar />

      <div className="container mt-4">
        <Routes>
          {/* Feed Page - Public */}
          <Route path="/" element={<Feed />} />

          {/* Create Post - Protected */}
          <Route
            path="/create"
            element={
              <ProtectedRoute>
                <CreatePost />
              </ProtectedRoute>
            }
          />

          {/* Login / Register - Public */}
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />

          {/* Catch All */}
          <Route path="*" element={<h3 className="text-center mt-5 text-danger">404 Not Found</h3>} />
        </Routes>
      </div>
    </Router>
  );
}

export default App;




import React from "react";
import { NavLink, useNavigate } from "react-router-dom";

function Navbar() {
  const navigate = useNavigate();
  const token = localStorage.getItem("token");

  const handleLogout = () => {
    localStorage.removeItem("token");
    localStorage.removeItem("userId");
    navigate("/login");
  };

  return (
    <nav className="navbar navbar-expand-lg navbar-dark bg-primary">
      <div className="container">
        {/* Brand */}
        <span
          className="navbar-brand fw-bold"
          style={{ cursor: "pointer" }}
          onClick={() => navigate("/")}
        >
          SocialNet
        </span>

        {/* Toggler for mobile */}
        <button
          className="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#mainNav"
        >
          <span className="navbar-toggler-icon"></span>
        </button>

        {/* Links */}
        <div className="collapse navbar-collapse" id="mainNav">
          <ul className="navbar-nav me-auto">
            <li className="nav-item">
              <NavLink end to="/" className="nav-link">
                Feed
              </NavLink>
            </li>

            {token && (
              <li className="nav-item">
                <NavLink to="/create" className="nav-link">
                  Create Post
                </NavLink>
              </li>
            )}
          </ul>

          <ul className="navbar-nav ms-auto">
            {!token ? (
              <>
                <li className="nav-item">
                  <NavLink to="/login" className="nav-link">
                    Login
                  </NavLink>
                </li>
                <li className="nav-item">
                  <NavLink to="/register" className="nav-link">
                    Register
                  </NavLink>
                </li>
              </>
            ) : (
              <li className="nav-item">
                <button className="btn btn-sm btn-warning" onClick={handleLogout}>
                  Logout
                </button>
              </li>
            )}
          </ul>
        </div>
      </div>
    </nav>
  );
}

export default Navbar;










import React, { useState, useEffect } from "react";
import { toggleLike } from "../api/postApi";
import { createComment, getComments } from "../api/commentApi";
import { sharePost, getShares } from "../api/shareApi";

function PostCard({ post, currentUser, refreshPosts }) {
  const [comment, setComment] = useState("");
  const [comments, setComments] = useState([]);
  const [shareCount, setShareCount] = useState(0);

  useEffect(() => {
    fetchComments();
    fetchShares();
  }, []);

  const fetchComments = async () => {
    try {
      const res = await getComments(post._id);
      setComments(res.data.comments || []);
    } catch (err) {
      console.error("Failed to load comments", err);
    }
  };

  const fetchShares = async () => {
    try {
      const res = await getShares(post._id);
      setShareCount(res.data.shares.length);
    } catch (err) {
      console.error("Failed to load shares", err);
    }
  };

  const handleLike = async () => {
    try {
      await toggleLike(post._id, currentUser._id);
      refreshPosts(); // to update like count
    } catch (err) {
      console.error("Error liking post", err);
    }
  };

  const handleComment = async (e) => {
    e.preventDefault();
    if (!comment.trim()) return;
    try {
      await createComment(post._id, comment);
      setComment("");
      fetchComments();
    } catch (err) {
      console.error("Error posting comment", err);
    }
  };

  const handleShare = async () => {
    try {
      await sharePost(post._id, currentUser._id);
      alert("Post shared!");
      fetchShares(); // update share count
    } catch (err) {
      console.error("Error sharing post", err);
    }
  };

  return (
    <div className="card mb-4 shadow-sm">
      <div className="card-body">
        <h5 className="card-title">{post?.creatorName || "Unknown"}</h5>
        <p className="card-text">{post.desc}</p>

        {/* 🔢 Counts */}
        <div className="d-flex justify-content-around text-muted mb-2">
          <span>❤️ {post.likes?.length || 0} Likes</span>
          <span>💬 {comments.length} Comments</span>
          <span>🔁 {shareCount} Shares</span>
        </div>

        {/* Buttons */}
        <div className="d-flex justify-content-around mb-3">
          <button className="btn btn-outline-primary" onClick={handleLike}>👍 Like</button>
          <button className="btn btn-outline-success" onClick={handleShare}>🔁 Share</button>
        </div>

        {/* Comment form */}
        <form onSubmit={handleComment} className="d-flex mb-2">
          <input
            className="form-control me-2"
            placeholder="Write a comment..."
            value={comment}
            onChange={(e) => setComment(e.target.value)}
          />
          <button className="btn btn-secondary" type="submit">Post</button>
        </form>

        {/* 🗨️ All Comments */}
        <div>
          {comments.map((cmt) => (
            <div key={cmt._id} className="mb-1">
              <strong>{cmt.creatorName || "User"}:</strong> {cmt.comment}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default PostCard;


import axios from "axios";
const BASE_URL = "http://localhost:5000/api/posts";

export const createPost = async (postData) => {
  return await axios.post(`${BASE_URL}/create`, postData);
};

export const getAllPosts = async () => {
  return await axios.get(BASE_URL);
};

export const toggleLike = async (postId, userId) => {
  return await axios.post(`${BASE_URL}/${postId}/like`, { userId });
};

import axios from "axios";
const BASE_URL = "http://localhost:5000/api/comments";

export const createComment = async (postId, commentText) => {
  return await axios.post(`${BASE_URL}/${postId}`, { comment: commentText });
};

export const getComments = async (postId) => {
  return await axios.get(`${BASE_URL}/${postId}`);
};

import axios from "axios";
const BASE_URL = "http://localhost:5000/api/share";

export const sharePost = async (postId, userId) => {
  return await axios.post(`${BASE_URL}/${postId}`, { userId });
};

export const getShares = async (postId) => {
  return await axios.get(`${BASE_URL}/${postId}`);
};


import React, { useState, useEffect } from "react";
import { toggleLike } from "../api/postApi";
import { createComment, getComments } from "../api/commentApi";
import { sharePost, getShares } from "../api/shareApi";

function PostCard({ post, currentUser, refreshPosts }) {
  const [comment, setComment] = useState("");
  const [comments, setComments] = useState([]);
  const [shareCount, setShareCount] = useState(0);

  useEffect(() => {
    fetchComments();
    fetchShares();
  }, []);

  const fetchComments = async () => {
    try {
      const res = await getComments(post._id);
      setComments(res.data.comments || []);
    } catch (err) {
      console.error("Failed to load comments", err);
    }
  };

  const fetchShares = async () => {
    try {
      const res = await getShares(post._id);
      setShareCount(res.data.shares.length);
    } catch (err) {
      console.error("Failed to load shares", err);
    }
  };

  const handleLike = async () => {
    try {
      await toggleLike(post._id, currentUser._id);
      refreshPosts(); // to update like count
    } catch (err) {
      console.error("Error liking post", err);
    }
  };

  const handleComment = async (e) => {
    e.preventDefault();
    if (!comment.trim()) return;
    try {
      await createComment(post._id, comment);
      setComment("");
      fetchComments();
    } catch (err) {
      console.error("Error posting comment", err);
    }
  };

  const handleShare = async () => {
    try {
      await sharePost(post._id, currentUser._id);
      alert("Post shared!");
      fetchShares(); // update share count
    } catch (err) {
      console.error("Error sharing post", err);
    }
  };

  return (
    <div className="card mb-4 shadow-sm">
      <div className="card-body">
        <h5 className="card-title">{post?.creatorName || "Unknown"}</h5>
        <p className="card-text">{post.desc}</p>

        {/* 🔢 Counts */}
        <div className="d-flex justify-content-around text-muted mb-2">
          <span>❤️ {post.likes?.length || 0} Likes</span>
          <span>💬 {comments.length} Comments</span>
          <span>🔁 {shareCount} Shares</span>
        </div>

        {/* Buttons */}
        <div className="d-flex justify-content-around mb-3">
          <button className="btn btn-outline-primary" onClick={handleLike}>👍 Like</button>
          <button className="btn btn-outline-success" onClick={handleShare}>🔁 Share</button>
        </div>

        {/* Comment form */}
        <form onSubmit={handleComment} className="d-flex mb-2">
          <input
            className="form-control me-2"
            placeholder="Write a comment..."
            value={comment}
            onChange={(e) => setComment(e.target.value)}
          />
          <button className="btn btn-secondary" type="submit">Post</button>
        </form>

        {/* 🗨️ All Comments */}
        <div>
          {comments.map((cmt) => (
            <div key={cmt._id} className="mb-1">
              <strong>{cmt.creatorName || "User"}:</strong> {cmt.comment}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

export default PostCard;


import axios from "axios";
const BASE_URL = "http://localhost:5000/api/posts";

export const createPost = async (postData) => {
  return await axios.post(`${BASE_URL}/create`, postData);
};

export const getAllPosts = async () => {
  return await axios.get(BASE_URL);
};

export const toggleLike = async (postId, userId) => {
  return await axios.post(`${BASE_URL}/${postId}/like`, { userId });
};

import axios from "axios";
const BASE_URL = "http://localhost:5000/api/comments";

export const createComment = async (postId, commentText) => {
  return await axios.post(`${BASE_URL}/${postId}`, { comment: commentText });
};

export const getComments = async (postId) => {
  return await axios.get(`${BASE_URL}/${postId}`);
};

import axios from "axios";
const BASE_URL = "http://localhost:5000/api/share";

export const sharePost = async (postId, userId) => {
  return await axios.post(`${BASE_URL}/${postId}`, { userId });
};

export const getShares = async (postId) => {


 import React, { useState } from 'react';
import axios from 'axios';

function Login({ onLogin }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async (e) => {
    e.preventDefault();
    try {
      const res = await axios.post("http://localhost:5000/api/users/login", { email, password });
      localStorage.setItem("token", res.data.token);
      onLogin(res.data); // update parent
      alert("Login successful!");
    } catch (err) {
      alert("Login failed");
    }
  };

  return (
    <form onSubmit={handleLogin}>
      <h3>Login</h3>
      <input type="email" value={email} placeholder="Email" onChange={(e) => setEmail(e.target.value)} required />
      <input type="password" value={password} placeholder="Password" onChange={(e) => setPassword(e.target.value)} required />
      <button type="submit">Login</button>
    </form>
  );
}

export default Login; 


import React, { useState } from 'react';
import axios from 'axios';

function Register() {
  const [form, setForm] = useState({
    fullName: '', email: '', password: '', confirmPassword: ''
  });

  const handleChange = (e) => {
    setForm({ ...form, [e.target.name]: e.target.value });
  };

  const handleRegister = async (e) => {
    e.preventDefault();
    try {
      await axios.post("http://localhost:5000/api/users/register", form);
      alert("Registration successful! You can now login.");
    } catch (err) {
      alert("Registration failed");
    }
  };

  return (
    <form onSubmit={handleRegister}>
      <h3>Register</h3>
      <input name="fullName" placeholder="Full Name" onChange={handleChange} required />
      <input name="email" type="email" placeholder="Email" onChange={handleChange} required />
      <input name="password" type="password" placeholder="Password" onChange={handleChange} required />
      <input name="confirmPassword" type="password" placeholder="Confirm Password" onChange={handleChange} required />
      <button type="submit">Register</button>
    </form>
  );
}

export default Register;
