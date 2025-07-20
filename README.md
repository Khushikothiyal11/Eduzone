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
