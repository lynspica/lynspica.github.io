---
layout: default
permalink: /blog/
title: blog
nav: true
nav_order: 1
---

<div class="post">

  {% assign blog_name_size = site.blog_name | size %}
  {% assign blog_description_size = site.blog_description | size %}

  {% if blog_name_size > 0 or blog_description_size > 0 %}
  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
  {% endif %}

  <div class="clearfix">
    <p style="margin-bottom: 20px;">
      Latest notes from <a href="https://lynspica.substack.com" target="_blank">lynspica.substack.com</a>
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
        // --- 1. Create the List Item (li) ---
        const listItem = document.createElement('li');
        listItem.style.marginBottom = "40px";

        // --- 2. Create Title (h3 > a) ---
        const titleHeader = document.createElement('h3');
        const titleLink = document.createElement('a');
        titleLink.className = "post-title"; // Uses theme's native CSS class
        titleLink.href = post.link;
        titleLink.target = "_blank";
        titleLink.innerText = post.title;
        titleHeader.appendChild(titleLink);

        // --- 3. Create Description (p) ---
        const desc = document.createElement('p');
        // Clean up HTML tags from description for a clean preview
        const cleanDesc = post.description.replace(/<[^>]*>?/gm, '').substring(0, 200) + "...";
        desc.innerText = cleanDesc;

        // --- 4. Create Meta Data (Date | Source) ---
        const meta = document.createElement('p');
        meta.className = "post-meta"; // Uses theme's native CSS class
        
        // Calculate read time (rough estimate)
        const readTime = Math.ceil(cleanDesc.length / 200); 
        const dateObj = new Date(post.pubDate);
        const dateStr = dateObj.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
        
        meta.innerHTML = `${readTime} min read &nbsp; &middot; &nbsp; ${dateStr} &nbsp; &middot; &nbsp; Substack`;

        // --- 5. Assemble the card ---
        listItem.appendChild(titleHeader);
        listItem.appendChild(desc);
        listItem.appendChild(meta);

        // --- 6. Add to page ---
        container.appendChild(listItem);
      });
    })
    .catch(error => {
      console.error('Error fetching Substack:', error);
      document.getElementById('substack-feed-list').innerHTML = 
        '<p>Visit <a href="https://lynspica.substack.com">lynspica.substack.com</a> to read the blog.</p>';
    });
</script>