# 📌 Project Notes – Automating Bluesky Posts with n8n (STAR Method)

## ⭐ Situation
Manually posting updates from newsletters or blogs to Bluesky was time-consuming and error-prone.  
The goal was to **automate the posting process** so every new RSS feed item from *First AI Movers* is automatically published as a native Bluesky post.

---

## 🎯 Task
- Build an **automation workflow in n8n**.  
- Fetch new items from an RSS feed.  
- Post them to Bluesky with title + link + image + hashtag.  
- Ensure reliability and error handling.  

---

## ⚙️ Action
Steps taken to implement the solution:

1. **RSS Feed Trigger**
   - Added `RSS Feed Read Trigger` node.  
   - Configured with:  
     - Feed URL: `https://rss.beehiiv.com/feeds/i8rpEDqA4c.xml`  
     - Polling time: every morning at 9 AM.
    
2. **Authentication**
   - Added `HTTP Request` node.  
   - Configured to call:  
     ```
     POST https://bsky.social/xrpc/com.atproto.server.createSession
     ```  

   - Sent body:  
     ```json
      {
      "identifier": "yourhandle.bsky.social",
      "password": "your_app_password"
      }
           
     ```
  
3. **Upload Image**
   - Added `HTTP Request` node.  
   - Configured to call:  
     ```
     GET {{ $('RSS Feed Trigger').item.json.enclosure.url }}
     ```
   - Authentication: None

   - Request Body: Not required

   - Options:
      - Response Format: File
      - Put Output in Field: data

4. **Download Image**
   - Added `HTTP Request` node.  
   - Configured to call:  
     ```
     POST https://bsky.social/xrpc/com.atproto.repo.uploadBlob
     ```
   - Sent header:
     ```json
     {"Authorization": "Bearer {{$json["accessJwt"]}}",
      "Content-Type": "Multipart/Form-Data"}
           
     ```
   - Request Body:
         - Body content type - n8n Binary File
         - Input Data Field Name - data


5. **Prepare Post Text**
   - Added `HTTP Request` node. 
   - Configured to call: 
     ```
     POST https://bsky.social/xrpc/com.atproto.repo.createRecord
     ```
   - Sent header:
     ```json
      {"Authorization": "Bearer {{ $('Authentication').item.json.accessJwt }}",
      "Content-Type": "application/json"}
     ```
    - Request body:
        ```json
          {
      "repo": "did:plc:YOUR_DID_HERE",
      "collection": "app.bsky.feed.post",
      "record": {
      "$type": "app.bsky.feed.post",
      "text": "{{ $('RSS Feed Trigger').item.json.title }}. Read more on First AI Movers: {{ $('RSS Feed Trigger').item.json.link }} #FirstAIMovers",
      "createdAt": "{{$now}}",
      "facets": [
      {
        "index": {
          "byteStart": {{ $('RSS Feed Trigger').item.json.title.length + '. Read more on First AI Movers: '.length }},
          "byteEnd": {{ $('RSS Feed Trigger').item.json.title.length + '. Read more on First AI Movers: '.length + $('RSS Feed Trigger').item.json.link.length }}
        },
        "features": [
          {
            "$type": "app.bsky.richtext.facet#link",
            "uri": "{{ $('RSS Feed Trigger').item.json.link }}"
          }
        ]
      }
      ],
      "embed": {
      "$type": "app.bsky.embed.images",
      "images": [
        {
          "alt": "{{ $('RSS Feed Trigger').item.json.title }}",
          "image": {
            "$type": "blob",
            "ref": {
              "$link": "{{ $('Upload Blob').item.json.blob.ref.$link }}"
            },
          "mimeType": "{{ $json.blob.mimeType }}",
          "size": {{ $json.blob.size }}
          },
          "aspectRatio": {
            "width": 1280,
            "height": 760
          }
        }
      ]
      }
      }
      }

        ```

5. **Test & Deploy**
   - Ran tests with sample posts.  
   - Deployed workflow to run automatically.  

---

## 🚀 Result
- Posts are now **automatically published** on Bluesky every time a new RSS feed item appears.  
- Saved manual effort & ensured **consistent updates**.  
- Workflow is reusable and can be adapted for other feeds in the future.  

---
