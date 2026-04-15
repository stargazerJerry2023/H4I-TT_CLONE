# Workshop: Fetching Videos from an API (In Progress)

## What are we building?

In this step, we are creating a **server-side function** that fetches videos from the **Pexels API**.

Right now, this version is a **first implementation**, meaning:
- the API call is working  
- the data is being fetched and mapped  
- but the function is not fully dynamic yet  

---

## File Setup

We are working in:

getvideo.tsx  

This file is responsible for:
- connecting to the Pexels API  
- retrieving video data  
- returning a simplified version of that data  

---

## Step 1: Enable Server-Side Execution

We start by adding:

```ts
"use server";
```

This tells Next.js that this function runs on the **backend**, not the frontend.

Why this matters:
- protects your API key  
- keeps sensitive logic off the client  

---

## Step 2: Import Types and API Key

```ts
import { VideoResponse } from "@/types/backend/types";

const API_KEY = process.env.API_KEY;
```

What’s happening:

- VideoResponse → type for API data (not fully used yet)  
- API_KEY → pulled from environment variables  

---

## Step 3: Create the Fetch Function

```ts
const GetVideos = async (
  type: string,
  query: string | null,
  pages: number,
  per_page: number = 5
) => {
```

Parameters:

- type → intended for future use (not used yet)  
- query → search term (not used yet)  
- pages → page number (not used yet)  
- per_page → number of results (default = 5, not used yet)  

Note: These parameters are defined but not yet implemented in the API call.

---

## Step 4: Validate API Key

```ts
if (!API_KEY || API_KEY == "") {
  throw new Error("missing API key");
}
```

This ensures the function does not run without a valid API key.

---

## Step 5: Fetch Data from the API

```ts
const res = await (
  await fetch(
    "https://api.pexels.com/v1/videos/search?query=nature&page=1&per_page=1",
    { headers: { Authorization: API_KEY } }
  )
).json();
```

What’s happening:

- Sends a request to the Pexels API  
- Uses a hardcoded query ("nature")  
- Always fetches:
  - page = 1  
  - per_page = 1  

This is a temporary setup — later we will replace this with dynamic values.

---

## Step 6: Map the Response Data

```ts
const videos = res.videos.map((video) => {
  return {
    id: video.id,
    width: video.id,
    height: video.height,
    url: video.video_files[0].link,
    duration: video.duration ? video.duration : 0,
    size: video.size,
    user: {
      id: video.user?.id,
      name: video.user?.name,
      url: video.user?.url,
    },
  };
});
```

What we’re doing:

- looping through res.videos  
- creating a simplified object for each video  
- extracting key fields:
  - id  
  - dimensions  
  - video URL  
  - duration  
  - user info  

Note:
- width is incorrectly set to video.id (should be video.width)  
- video_files[0] is used without checking if it exists  

---

## Step 7: Return the Data

```ts
return videos;
```

We return only the array of videos, not the full API response.

---

  
