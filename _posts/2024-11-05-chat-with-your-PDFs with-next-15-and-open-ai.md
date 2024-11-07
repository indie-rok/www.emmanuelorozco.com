---
layout: post
title: Chat with your PDFs with Next 15 and OpenAI
date: 2024-11-05 18:30:00 +0000
tags:
  - javascript
  - nextjs
  - tutorial
---

## Table of Contents

1. [Intro](#intro)
2. [Overview](#overview)
   - [Client Side](#client-side)
   - [Server Side](#server-side)
3. [Frontend](#frontend)
4. [Backend](#backend)
5. [Deployment to Vercel](#deployment-to-vercel)

---

### 1. Intro

I am building a tool to read a CV and suggest improvements using OpenAI's API.

The only problem: To use OpenAI's computer vision, **we need to provide images** (in base 64), not the PDF file.

The first natural approach is to convert the PDF (to image) on the server, however, because [Vercel Limitations](https://vercel.com/docs/functions/limitations) it's not possible to install system libraries in edge functions (like pandoc).

So plan B: Do the conversion client-side, and let's be honest, PDF processing client-side can be a bit tedious, so I am writing this tutorial for you, my dear stuck dev ✅

You can check the whole code repo [here](https://github.com/indie-rok/pdf-tools-nextjs-15)

---

### 2. Overview

This is how the app is architected:

![app architecture](https://emmanuelorozco.com/assets/blog/diagram-chat-pdf.jpg)

Client Side

1. Read image from file input
2. Convert File Object to ArrayBuffer
3. Convert ArrayBuffer to PDFJS Object (for processing)
4. For each page:
   a. Draw the PDF in an image in an HTML canvas (scale 1 and with a viewport)
   b. Convert the canvas into a base64 image
5. Save to an array
6. Send base64 images via POST JSON

Server Side

1. Receive the base64 via POST JSON
2. Send the request to OpenAI with the base 64 image
3. Receive and display the result.

---

### 3. Frontend

So, let's start with the front end, we will add an input, a button, and the logic to convert the PDF to images (client-side) and to send those images to the client:

```js
// app/page.js
"use client";

import React, { useState } from "react";
import { version, GlobalWorkerOptions, getDocument } from "pdfjs-dist";

const workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${version}/pdf.worker.min.mjs`;
GlobalWorkerOptions.workerSrc = workerSrc;

const PDFToImage = () => {
  const [images, setImages] = useState([]);
  const [resume, setResume] = useState("");

  const handlePDFUpload = async (event) => {
    const file = event.target.files?.[0]; // read PDF
    const fileReader = new FileReader();
    fileReader.readAsArrayBuffer(file); // convert to array buffer
    fileReader.onload = async (fileAsArrayBuffer) => {
      const pdf = await getDocument(fileAsArrayBuffer.target.result).promise; // convert array buffer to pdf object

      const imagesArray = [];
      for (let i = 0; i < pdf.numPages; i++) {
        // for each page
        const page = await pdf.getPage(i + 1); // get page
        const viewport = page.getViewport({ scale: 1 });
        const canvas = document.createElement("canvas");
        const context = canvas.getContext("2d");
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        await page.render({ canvasContext: context, viewport }).promise; // draw pdf to canvas image
        imagesArray.push(canvas.toDataURL("image/png")); // convert canvas to img
      }
      setImages(imagesArray);
    };
  };

  const handleSubmit = async () => {
    try {
      const response = await fetch("/api/parse_pdf", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ images }),
      });

      const data = await response.json();
      setResume(data.resume);
    } catch (error) {
      console.error("Error:", error);
    }
  };

  return (
    <>
      <h1>Upload Your PDF</h1>
      <pre>{resume}</pre>
      <div>
        <input type="file" accept=".pdf" onChange={handlePDFUpload} />

        <button onClick={handleSubmit}>Submit</button>

        {images.map((i, key) => (
          <div key={key}>
            <img width={400} src={i} />
          </div>
        ))}
      </div>
    </>
  );
};

export default PDFToImage;
```

A few elements to highlight:

1. We need a global worker for pdf-js to work from a CDN, it does not work with the local build.
2. We use the browser file reader API to handle the conversion from file to ArrayBuffer.
3. You need to set the scale of the canvas viewport to at least one, otherwise, it will not work.

Once we send the images to the client, we just need to process them on the backend.

---

### 4. Backend

```js
// api/parse_pdf/route.js

import { NextResponse } from "next/server";

function generatePrompt(images) {
  return [
    {
      role: "user",
      content: [
        { type: "text", text: "What’s in this image?" },
        ...images.map((url) => ({
          type: "image_url",
          image_url: {
            url,
          },
        })),
      ],
    },
  ];
}

export async function POST(req) {
  try {
    const { images } = await req.json();

    const messages = generatePrompt(images);

    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${OPENAI_API_KEY}`,
      },
      body: JSON.stringify({ messages, model: "gpt-4o" }),
    });

    const data = await response.json();

    return NextResponse.json({ resume: data.choices[0].message.content });
  } catch (error) {
    console.log(error);
    return NextResponse.json(
      { error: "Internal Server Error" },
      { status: 500 }
    );
  }
}
```

A few elements to highlight:

1. We don't need to install any OpenAI library, we can just send a POST request directly.
2. We need to convert our base 64 images to a format recognized by OpenAI (generatePrompt function)

---

### 5. Deployment to Vercel

After creating and deploying the project to Vercel, There is one missing problem, this error:

```bash
TypeError: Promise.withResolvers is not a function
```

The problem is that pdfjs uses Promise.withResolvers, which is part of Node 22 > and Vercel is running Node 20.

So let's add this missing method with a [polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill), in your root app layout (or your top-level component):

```js
if (typeof Promise.withResolvers === "undefined") {
  Promise.withResolvers = function () {
    let resolve, reject;
    const promise = new Promise((res, rej) => {
      resolve = res;
      reject = rej;
    });
    return { promise, resolve, reject };
  };
}
```

After this, the deployment is working.

Hope it helped!

[insert gift]
