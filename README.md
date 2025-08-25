# bluesky-rss-n8n
Automated workflow to post RSS feed items to Bluesky using n8n.


# Automating Bluesky Posts from RSS with n8n

This project shows how to **automatically post RSS feed items to Bluesky** using [n8n](https://n8n.io).  
It fixes the common problem where links posted via Make.com or other tools appear as plain text (not clickable).  
With this workflow, every RSS feed item becomes a **native Bluesky post with clickable links, title, hashtags, and images**.

---

## STAR Method Documentation

### **Situation**
- **Problem:** Posting RSS feed items from [First AI Movers](https://bsky.app/profile/firstaimovers.bsky.social) to Bluesky.  
- **Issue:** Most automation tools (e.g., Make.com) only post plain text — links show as non-clickable.  
- **Cause:** Bluesky requires *rich text facets* in the API payload to render clickable links.  
- **Goal:** Build a fully automated workflow in n8n that posts RSS items as proper Bluesky posts with links, hashtags, and preview images.

---

### **Task**
- Create an **n8n pipeline** that:
  1. Watches an RSS feed for new items.
  2. Authenticates with Bluesky via the ATProto API.
  3. Downloads and uploads images to Bluesky.
  4. Posts text + clickable link + image + hashtag to Bluesky.

---

### **Action**
Steps taken:
1. **RSS Feed Trigger**  
   Watches new articles from Beehiiv RSS:  
   `https://rss.beehiiv.com/feeds/i8rpEDqA4c.xml`

2. **Authenticate with Bluesky**  
   - POST to `com.atproto.server.createSession`
   - Uses handle + App Password (generated in Bluesky Settings).

3. **Download & Upload Image**  
   - Download the article’s enclosure image.
   - Upload to Bluesky with `com.atproto.repo.uploadBlob`.

4. **Create Post with Facets**  
   - POST to `com.atproto.repo.createRecord`
   - Compose text:  
     ```
     {title}. Read more on First AI Movers: {link} #FirstAIMovers
     ```
   - Add a **facet** object marking the link range:
     ```json
     "facets": [
       {
         "index": {
           "byteStart": 56,
           "byteEnd": 109
         },
         "features": [
           {
             "$type": "app.bsky.richtext.facet#link",
             "uri": "https://example.com"
           }
         ]
       }
     ]
     ```

---

### **Result**
- ✅ New RSS stories are auto-posted to Bluesky within minutes.  
- ✅ Links are **blue and clickable** (solving the Make.com limitation).  
- ✅ Each post includes:
  - Title (as hook text)  
  - Clickable link  
  - Hashtag `#FirstAIMovers`  
  - Image preview  

Example Post:
<img width="400" height="295" alt="image" src="https://github.com/user-attachments/assets/0ef67f7b-bc0c-470c-83cc-7ec3824620d2" />



