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

## What are we building?

In this step, we are creating a **frontend video component** that displays videos and handles user interaction.

This component:
- renders a video using a URL
- listens for user clicks
- shows a play/pause animation overlay

Think of this as the **UI layer that displays the videos fetched from our backend**.

---

## File Setup

We are working in:

VideoFeed.tsx  

This file is responsible for:
- displaying a video  
- handling user interaction (clicks)  
- showing UI feedback (play icon)  

---

## Step 1: Enable Client-Side Execution

```ts
"use client";
```

This tells Next.js that this component runs on the **frontend (browser)**.

Why this matters:
- allows use of React hooks like useState and useEffect  
- enables interactivity (clicks, animations, etc.)  

---

## Step 2: Import Hooks and Types

```ts
import { VideoResponse } from "@/types/backend/types";
import { useCallback, useEffect, useRef, useState } from "react";
```

What’s happening:

- useState → manages component state  
- useRef → references the video element directly  
- useEffect / useCallback → imported but not used yet  
- VideoResponse → imported but not used in this component  

Note: Some imports are currently unused and can be cleaned up later.

---

## Step 3: Create the Component

```ts
export default function VideoFeed({ url }: { url: string }) {
```

This component takes in:

- url → the video source to display  

---

## Step 4: Reference the Video Element

```ts
const videoRef = useRef<HTMLVideoElement>(null);
```

This allows us to:
- directly access the video DOM element  
- control playback (play/pause)  

---

## Step 5: Create State for Play/Pause UI

```ts
const [isPaused, setPaused] = useState<{
  type: boolean,
  id: number
} | null>(null);
```

What this state does:

- type → determines whether we show play or pause icon  
- id → ensures animation re-renders (used as a key)  
- null → no icon shown initially  

---

## Step 6: Handle Click Interaction

```ts
const togglePlay = (e: React.MouseEvent<HTMLDivElement>) => {
  e.stopPropagation();

  if (videoRef.current) {
    if (videoRef.current.paused) {
      setPaused({ type: false, id: Date.now() });
    }
  }
};
```

What’s happening:

- stops click from bubbling up  
- checks if the video exists  
- checks if the video is paused  
- updates state to show the play icon  

Note:
- This currently does NOT actually play or pause the video  
- It only updates the UI state  

---

## Step 7: Render the Video

```tsx
<video
  ref={videoRef}
  src={url}
  className="z-0 absolute inset-0 h-full object-cover rounded-2xl"
/>
```

What this does:

- renders the video using the provided URL  
- attaches the ref for control  
- styles the video to:
  - fill the container  
  - maintain aspect ratio  
  - have rounded corners  

---

## Step 8: Add Click Overlay

```tsx
<div
  className="absolute inset-0 z-10 w-full h-full flex justify-center items-center"
  onClick={togglePlay}
>
```

Why this exists:

- creates a clickable layer over the video  
- captures user interaction  
- centers UI elements (like play icon)  

---

## Step 9: Show Play/Pause Animation

```tsx
{isPaused && (
  <img
    key={isPaused.id}
    src={isPaused.type ? "/pause.png" : "/play.png"}
    className="w-[60px] h-[60px] animate-float-up pointer-events-none"
    alt={isPaused.type ? "Pause" : "Play"}
  />
)}
```

What’s happening:

- shows an icon when state is set  
- switches between play and pause image  
- uses key to trigger animation  
- animation class creates a floating effect  

---

## Current Code

```tsx
"use client";
import { VideoResponse } from "@/types/backend/types";
import { useCallback, useEffect, useRef, useState } from "react";

export default function VideoFeed({url}: {url: string}) {
  const videoRef= useRef<HTMLVideoElement>(null)

  const [isPaused, setPaused]= useState<{
    type: boolean,
    id: number
  } | null>(null);

  const togglePlay = (e: React.MouseEvent<HTMLDivElement>)=> {
    e.stopPropagation()
    if (videoRef.current){
      if (videoRef.current.paused){
        setPaused({type: false, id: Date.now()})
      }
    }
  }

  return (
    <>
      <div className="h-full w-full overflow-hidden relative flex justify-center items-center">
        <video
          ref={videoRef}
          src={url}
          className="z-0 absolute inset-0 h-full object-cover rounded-2xl"
        />

        <div
          className="absolute inset-0 z-10 w-full h-full flex justify-center items-center"
          onClick={togglePlay}
        >
          {isPaused && (
            <img
              key={isPaused.id}
              src={isPaused.type ? "/pause.png" : "/play.png"}
              className="w-[60px] h-[60px] animate-float-up pointer-events-none"
              alt={isPaused.type ? "Pause" : "Play"}
            />
          )}
        </div>
      </div>
    </>
  );
}
```

---

---
## What are we building?

In this step, we are connecting our **backend video fetch function** to our **frontend video component**.

This page:
- fetches video data from the backend  
- passes that data into the VideoFeed component  
- renders the video on the screen  

Think of this as the **bridge between your backend and UI**.

---

## File Setup

We are working in:

app/page.tsx  

This file is responsible for:
- fetching data on the server  
- rendering the main UI  
- passing props to components  

---

## Step 1: Enable Server-Side Execution

```ts
"use server";
```

This ensures that the code runs on the **server**, allowing us to:
- safely call backend functions  
- keep API calls secure  

---

## Step 2: Import Components and Functions

```ts
import VideoFeed from "@/component/Video";
import GetVideos from "@/lib/getVideos";
```

What’s happening:

- VideoFeed → frontend component that displays a video  
- GetVideos → backend function that fetches video data  

---

## Step 3: Create the Page Component

```ts
export default async function Home() {
```

This is an **async server component**, meaning:
- we can fetch data directly inside it  
- no need for useEffect or client-side fetching  

---

## Step 4: Fetch Video Data

```ts
const data = await GetVideos("", "", 0, 0);
```

What’s happening:

- calling our backend function to retrieve videos  
- passing placeholder values for now  

Note:
- parameters are not being used yet  
- this is just to confirm the data flow works  

---

## Step 5: Debug the Data

```ts
console.log(data[0].url);
```

This logs the first video URL to confirm:
- data is being fetched correctly  
- the structure is what we expect  

---

## Step 6: Render the UI

```tsx
return (
  <main className="h-screen w-screen grid place-items-center">
    <p className="text-lg">Workshop TODO: Implement app/page.tsx</p>
    <VideoFeed url={data[0].url} />
  </main>
);
```

What’s happening:

- creates a full-screen layout  
- displays placeholder text  
- passes the first video’s URL into VideoFeed  

---

## Current Code

```tsx
"use server";

import VideoFeed from "@/component/Video";
import GetVideos from "@/lib/getVideos";

export default async function Home() {
  // TODO: Fetch initial videos from Pexels and pass them to <VideoFeed />
  // Suggested starter:
  // const data = await getVideosByQuery("space", 1, 5);
  // return <VideoFeed videoRes={data} />;

  const data = await GetVideos("", "", 0, 0);

  console.log(data[0].url);

  return (
    <main className="h-screen w-screen grid place-items-center">
      <p className="text-lg">Workshop TODO: Implement app/page.tsx</p>
      <VideoFeed url={data[0].url} />
    </main>
  );
}
```

---
  
