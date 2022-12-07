---
layout: articles
title: Blog
show_title: false
key: articles-item-cover-excerpt-readmore-info
cover: /docs/assets/images/axure/articles-item-cover-excerpt-readmore-info.jpg
articles:
  data_source: paginator.posts
  article_type: BlogPosting
  show_cover: true
  show_excerpt: true
  show_readmore: true
  show_info: true
---

<div class="layout--home">
  {%- include paginator.html -%}
</div>
<script>
  {%- include scripts/home.js -%}
</script>

{{ content }}
