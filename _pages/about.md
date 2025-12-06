---
layout: default
permalink: /blog/
title: blog
nav: true
nav_order: 1
---

<div class="post">

  {% assign blog_name_size = site.blog_name | size %}
  {% if blog_name_size > 0 %}
  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
  {% endif %}

  <div class="clearfix">
    <p style="margin-bottom: 40px; font-style: italic; border-bottom: 1px solid #eee; padding-bottom: 20px;">
      Writing from <a href="https://lynspica.substack.com" target="_blank" style="text-decoration: underline;">lynspica.substack.com</a>
    </p>
  </div>

  <ul class="post-list" id="substack-feed-list">
    <p>Loading posts...</p>
  </ul>

</div>

<script>
  const substackUrl = "https://lynspica.substack.com/feed";
  const apiUrl = `https://api.rss2json.com/v1/api.json?rss_url=${substackUrl}`;

  fetch(apiUrl)
    .then(response => response.json())
    .then(data => {
      const container = document.getElementById('substack-feed-list');
      const posts = data.items;
      
      // Clear "Loading..." message
      container.innerHTML = '';

      posts.forEach(post => {
        // --- Create List Item ---
        const listItem = document.createElement('li');
        listItem.style.marginBottom = "50px"; 

        // --- Title ---
        const titleHeader = document.createElement('h3');
        const titleLink = document.createElement('a');
        titleLink.className = "post-title"; 
        titleLink.href = post.link;
        titleLink.target = "_blank";
        titleLink.innerText = post.title;
        titleLink.style.textDecoration = "none";
        titleHeader.appendChild(titleLink);

        // --- Description (Body Snippet) ---
        // Since Substack doesn't send the subtitle, we take the first 400 chars of the body
        const desc = document.createElement('p');
        // 1. Strip HTML tags
        let cleanDesc = post.description.replace(/<[^>]*>?/gm, '');

        // 2. Cut to 400 chars, but back up to the last space so we don't chop a word
        if (cleanDesc.length > 400) {
          cleanDesc = cleanDesc.substring(0, 400);
          cleanDesc = cleanDesc.substring(0, cleanDesc.lastIndexOf(" ")) + "...";
        }        desc.innerText = cleanDesc;
        desc.style.marginTop = "10px";
        desc.style.lineHeight = "1.6";
        desc.style.color = "#444";

        // --- Metadata ---
        const meta = document.createElement('p');
        meta.className = "post-meta";
        const readTime = Math.ceil(cleanDesc.length / 200); 
        const dateObj = new Date(post.pubDate);
        const dateStr = dateObj.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
        
        meta.innerHTML = `${readTime} min read &nbsp; &middot; &nbsp; ${dateStr} &nbsp; &middot; &nbsp; Substack`;

        // --- Assemble ---
        listItem.appendChild(titleHeader);
        listItem.appendChild(desc);
        listItem.appendChild(meta);
        container.appendChild(listItem);
      });
    })
    .catch(error => {
      console.error('Error fetching Substack:', error);
      document.getElementById('substack-feed-list').innerHTML = 
        '<p>Visit <a href="https://lynspica.substack.com">lynspica.substack.com</a> to read the blog.</p>';
    });
</script>