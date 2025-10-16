---
title: 首页
date: 2023-01-01 00:00:00
layout: custom-home
---

## 🎯 最新文章

<%- list_posts({ limit: 5, order: -1 }) %>

## 📚 热门分类

<%- list_categories({
show_count: true,
limit: 8,
order: -1,
style: 'list'
}) %>

## 🔖 热门标签

<%- list_tags({
show_count: true,
limit: 15,
order: -1,
style: 'list'
}) %>