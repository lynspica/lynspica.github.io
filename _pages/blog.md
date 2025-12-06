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
      container.innerHTML = '';

      posts.forEach(post => {
        // --- 1. EXTRACT IMAGE (Improved) ---
        let imageUrl = post.thumbnail; 
        
        // If no thumbnail, check the "enclosure" (Standard RSS location for cover images)
        if (!imageUrl && post.enclosure && post.enclosure.link) {
          imageUrl = post.enclosure.link;
        }

        // If still empty, try to find an image tag in the content
        if (!imageUrl) {
           const imgMatch = post.content.match(/<img[^>]+src="([^">]+)"/);
           if (imgMatch) {
             imageUrl = imgMatch[1];
           }
        }

        // --- 2. CREATE THE CARD STRUCTURE ---
        const listItem = document.createElement('li');
        listItem.style.marginBottom = "40px";
        
        const rowDiv = document.createElement('div');
        rowDiv.className = "row"; 

        // --- 3. CREATE TEXT COLUMN ---
        // If image exists, text takes 9 columns. If not, it takes 12 (full width).
        const textCol = document.createElement('div');
        textCol.className = imageUrl ? "col-sm-9" : "col-sm-12";

        // Title
        const titleHeader = document.createElement('h3');
        const titleLink = document.createElement('a');
        titleLink.className = "post-title";
        titleLink.href = post.link;
        titleLink.target = "_blank";
        titleLink.innerText = post.title;
        titleHeader.appendChild(titleLink);

        // Description
        const desc = document.createElement('p');
        const cleanDesc = post.description.replace(/<[^>]*>?/gm, '').substring(0, 200) + "...";
        desc.innerText = cleanDesc;

        // Meta Data
        const meta = document.createElement('p');
        meta.className = "post-meta";
        const readTime = Math.ceil(cleanDesc.length / 200); 
        const dateObj = new Date(post.pubDate);
        const dateStr = dateObj.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
        meta.innerHTML = `${readTime} min read &nbsp; &middot; &nbsp; ${dateStr} &nbsp; &middot; &nbsp; Substack`;

        textCol.appendChild(titleHeader);
        textCol.appendChild(desc);
        textCol.appendChild(meta);
        rowDiv.appendChild(textCol);

        // --- 4. CREATE IMAGE COLUMN (Only if image found) ---
        if (imageUrl) {
          const imgCol = document.createElement('div');
          imgCol.className = "col-sm-3";
          
          const img = document.createElement('img');
          img.className = "card-img";
          img.src = imageUrl;
          img.style.objectFit = "cover";
          img.style.height = "100%";
          img.style.borderRadius = "4px"; 
          img.alt = "Post image";

          imgCol.appendChild(img);
          rowDiv.appendChild(imgCol);
        }

        // --- 5. ASSEMBLE ---
        listItem.appendChild(rowDiv);
        container.appendChild(listItem);
      });
    })
    .catch(error => {
      console.error('Error fetching Substack:', error);
      document.getElementById('substack-feed-list').innerHTML = 
        '<p>Visit <a href="https://lynspica.substack.com">lynspica.substack.com</a> to read the blog.</p>';
    });
</script>