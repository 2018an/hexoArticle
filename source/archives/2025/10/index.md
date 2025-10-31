---
title: 完整归档
layout: archives
comments: false
aside: false
---

<div class="advanced-archives">
    <!-- 统计信息 -->
    <div class="archive-stats">
        <div class="stat-item">
            <span class="stat-number"><%= site.posts.length %></span>
            <span class="stat-label">总文章数</span>
        </div>
        <div class="stat-item">
            <span class="stat-number"><%= site.tags.length %></span>
            <span class="stat-label">标签数</span>
        </div>
        <div class="stat-item">
            <span class="stat-number"><%= site.categories.length %></span>
            <span class="stat-label">分类数</span>
        </div>
    </div>

    <!-- 年份导航 -->
    <div class="year-nav">
        <% 
        const years = [];
        site.posts.sort('date', -1).each(function(post){
            const year = post.date.year();
            if (!years.includes(year)) {
                years.push(year);
            }
        });
        %>
        <% years.forEach(function(year){ %>
            <a href="#year-<%= year %>" class="year-link"><%= year %>年</a>
        <% }); %>
    </div>

    <!-- 归档内容 -->
    <div class="archive-content">
        <script>
        // 这里的内容由模板生成
        </script>
    </div>
</div>

<style>
.advanced-archives {
    max-width: 900px;
    margin: 0 auto;
    padding: 20px;
}

.archive-stats {
    display: flex;
    justify-content: space-around;
    margin: 30px 0;
    padding: 20px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 10px;
    color: white;
}

.stat-item {
    text-align: center;
}

.stat-number {
    display: block;
    font-size: 2em;
    font-weight: bold;
}

.stat-label {
    font-size: 0.9em;
    opacity: 0.9;
}

.year-nav {
    text-align: center;
    margin: 30px 0;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 10px;
}

.year-link {
    display: inline-block;
    margin: 0 10px;
    padding: 8px 16px;
    background: white;
    border-radius: 20px;
    text-decoration: none;
    color: #333;
    transition: all 0.3s ease;
}

.year-link:hover {
    background: #007cba;
    color: white;
    transform: translateY(-2px);
}
</style>
