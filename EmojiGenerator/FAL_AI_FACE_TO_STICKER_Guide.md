
# Using the `fal-ai/face-to-sticker` Model

## Introduction

The `fal-ai/face-to-sticker` model allows you to generate sticker images from face photos using AI. This guide will help you integrate this model into your applications using the `@fal-ai/serverless-client` library.

## Prerequisites

- **Node.js** installed on your system.
- An **API key** from [fal.ai](https://fal.ai).

## Installation

Install the `@fal-ai/serverless-client` package using your preferred package manager:

```bash
npm install --save @fal-ai/serverless-client
```

## Authentication

Set your API key as an environment variable:

```bash
export FAL_KEY="YOUR_API_KEY"
```

Alternatively, you can configure the API key directly in your code:

```javascript
import * as fal from "@fal-ai/serverless-client";

fal.config({
  credentials: "YOUR_API_KEY"
});
```

**Note:** Protect your API key. Do not expose it in client-side code.

## Generating an Image

Here's how to submit a request to generate a sticker image:

```javascript
import * as fal from "@fal-ai/serverless-client";

const result = await fal.subscribe("fal-ai/face-to-sticker", {
  input: {
    image_url: "https://example.com/your-face-image.jpg",
    prompt: "a person"
  },
  logs: true,
  onQueueUpdate: (update) => {
    if (update.status === "IN_PROGRESS") {
      update.logs.map((log) => log.message).forEach(console.log);
    }
  },
});

console.log(result);
```

## Handling Long-Running Requests

For requests that may take longer, you can use the queue methods:

### Submit a Request

```javascript
const { request_id } = await fal.queue.submit("fal-ai/face-to-sticker", {
  input: {
    image_url: "https://example.com/your-face-image.jpg",
    prompt: "a person"
  },
  webhookUrl: "https://your-server.com/webhook", // Optional
});
```

### Check Request Status

```javascript
const status = await fal.queue.status("fal-ai/face-to-sticker", {
  requestId: request_id,
  logs: true,
});
```

### Retrieve the Result

```javascript
const result = await fal.queue.result("fal-ai/face-to-sticker", {
  requestId: request_id
});
```

## File Handling

You can upload files directly if you don't have a public URL:

```javascript
const file = new File(["<binary data>"], "face.jpg", { type: "image/jpeg" });
const imageUrl = await fal.storage.upload(file);

const result = await fal.subscribe("fal-ai/face-to-sticker", {
  input: {
    image_url: imageUrl,
    prompt: "a person"
  },
});
```

## Input Parameters

- **image_url** (`string`): URL of the input face image.
- **prompt** (`string`): Description of the image you want to generate.
- **negative_prompt** (`string`, optional): Aspects you want to avoid in the generated image.
- **num_inference_steps** (`integer`, default: `20`): Number of inference steps.
- **guidance_scale** (`float`, default: `4.5`): Controls adherence to the prompt.
- **instant_id_strength** (`float`, default: `0.7`): Strength of the instant ID.
- **ip_adapter_weight** (`float`, default: `0.2`): Weight of the IP adapter.
- **ip_adapter_noise** (`float`, default: `0.5`): Amount of noise added.
- **image_size** (`string`, default: `"square_hd"`): Size of the generated image. Options include `"square_hd"`, `"square"`, `"portrait_4_3"`, `"portrait_16_9"`, `"landscape_4_3"`, `"landscape_16_9"`.
- **upscale** (`boolean`, default: `false`): Whether to upscale the image by 2x.
- **upscale_steps** (`integer`, default: `10`): Steps used for upscaling.
- **seed** (`integer`, optional): Seed for random number generation.
- **enable_safety_checker** (`boolean`, default: `true`): Enables or disables the safety checker.

## Output Schema

- **images** (`list<Image>`): List of generated images.
- **sticker_image** (`Image`): The generated sticker image.
- **sticker_image_background_removed** (`Image`): The sticker image with background removed.
- **seed** (`integer`): Seed used during inference.
- **has_nsfw_concepts** (`object`): Indicates if NSFW content was detected.

### Image Object Structure

- **url** (`string`): URL of the image.
- **content_type** (`string`): MIME type of the image.
- **file_name** (`string`): Name of the image file.
- **file_size** (`integer`): Size in bytes.
- **width** (`integer`): Width in pixels.
- **height** (`integer`): Height in pixels.

## Example

```javascript
import * as fal from "@fal-ai/serverless-client";

const result = await fal.subscribe("fal-ai/face-to-sticker", {
  input: {
    image_url: "https://example.com/your-face-image.jpg",
    prompt: "cartoon style",
    num_inference_steps: 30,
    guidance_scale: 7.5,
    image_size: "square_hd",
    upscale: true,
    enable_safety_checker: true,
  },
});

console.log(result.sticker_image.url);
```

## Conclusion

By following this guide, you can integrate the `fal-ai/face-to-sticker` model into your application to generate sticker images from face photos. Adjust the input parameters to fine-tune the output according to your needs.

For more information, refer to the [fal-ai documentation](https://fal.ai/docs).
