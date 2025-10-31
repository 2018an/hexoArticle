---
title: 标签集合
layout: tags
comments: false
aside: false
top_img: false
type: tags
---

<div class="tag-container">
  <!-- 标签云部分 -->
  <div class="tag-cloud-section">
    <h2>📁 标签云</h2>
    <div class="tag-cloud-content">
      <%- tagcloud({min_font: 14, max_font: 30, amount: 100, color: true, start_color: '#9745b5', end_color: '#e95420'}) %>
    </div>
  </div>

  <!-- 详细标签列表 -->
  <div class="tag-detail-section">
    <h2>📋 所有标签</h2>
    <div class="tag-detail-list">
      <% site.tags.sort('name').map(function(tag){ %>
        <div class="tag-group">
          <h3 class="tag-title" id="<%= tag.name %>">
            <span class="tag-icon">🏷️</span>
            <%= tag.name %>
            <span class="tag-badge"><%= tag.posts.length %> 篇文章</span>
          </h3>
          <div class="tag-posts">
            <% tag.posts.sort('date', -1).map(function(post){ %>
              <div class="tag-post-item">
                <time class="post-time"><%= post.date.format('MM-DD') %></time>
                <a class="post-link" href="<%- url_for(post.path) %>">
                  <%= post.title %>
                  <% if(post.top) { %>
                    <span class="top-flag">置顶</span>
                  <% } %>
                </a>
              </div>
            <% }); %>
          </div>
        </div>
      <% }); %>
    </div>
  </div>
</div>

<style>
.tag-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.tag-cloud-section {
  background: var(--card-bg);
  border-radius: 12px;
  padding: 30px;
  margin-bottom: 30px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.tag-cloud-content {
  text-align: center;
  line-height: 2.5;
}

.tag-detail-section {
  background: var(--card-bg);
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.tag-group {
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid var(--border-color);
}

.tag-group:last-child {
  border-bottom: none;
}

.tag-title {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
  color: var(--text-color);
}

.tag-badge {
  background: var(--theme-color);
  color: white;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
}

.tag-posts {
  display: grid;
  gap: 8px;
}

.tag-post-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 8px 0;
}

.post-time {
  color: var(--secondary-text-color);
  font-size: 14px;
  min-width: 60px;
}

.post-link {
  color: var(--text-color);
  text-decoration: none;
  transition: color 0.3s;
}

.post-link:hover {
  color: var(--theme-color);
}

.top-flag {
  background: #ff4757;
  color: white;
  font-size: 12px;
  padding: 1px 6px;
  border-radius: 4px;
  margin-left: 8px;
}
</style>