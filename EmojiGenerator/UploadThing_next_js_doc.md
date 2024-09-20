Next.js Pages Router Setup
No shame in using Page Router still! We still use it for a lot of things at Ping 😊. If your code is still mostly served from the pages/ directory, this is where you want to be

Setting up your environment
Install the packages
npm
pnpm
yarn
bun
npm install uploadthing @uploadthing/react

Copy
Copied!
Add env variables
UPLOADTHING_TOKEN=... # A token for interacting with the SDK

Copy
Copied!
If you don't already have a uploadthing secret key, sign up ↗ and create one from the dashboard! ↗

Set Up A FileRouter
Creating your first FileRoute
All files uploaded to uploadthing are associated with a FileRoute. The following is a very minimalistic example, with a single FileRoute "imageUploader". Think of a FileRoute similar to an endpoint, it has:

Permitted types ["image", "video", etc]
Max file size
(Optional) middleware to authenticate and tag requests
onUploadComplete callback for when uploads are completed
To get full insight into what you can do with the FileRoutes, please refer to the File Router API.

server/uploadthing.ts
import type { NextApiRequest, NextApiResponse } from "next";
import { createUploadthing, type FileRouter } from "uploadthing/next-legacy";
import { UploadThingError } from "uploadthing/server";
const f = createUploadthing();
const auth = (req: NextApiRequest, res: NextApiResponse) => ({ id: "fakeId" }); // Fake auth function
// FileRouter for your app, can contain multiple FileRoutes
export const ourFileRouter = {
  // Define as many FileRoutes as you like, each with a unique routeSlug
  imageUploader: f({ image: { maxFileSize: "4MB" } })
    // Set permissions and file types for this FileRoute
    .middleware(async ({ req, res }) => {
      // This code runs on your server before upload
      const user = await auth(req, res);
      // If you throw, the user will not be able to upload
      if (!user) throw new UploadThingError("Unauthorized");
      // Whatever is returned here is accessible in onUploadComplete as `metadata`
      return { userId: user.id };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      // This code RUNS ON YOUR SERVER after upload
      console.log("Upload complete for userId:", metadata.userId);
      console.log("file url", file.url);
      // !!! Whatever is returned here is sent to the clientside `onClientUploadComplete` callback
      return { uploadedBy: metadata.userId };
    }),
} satisfies FileRouter;
export type OurFileRouter = typeof ourFileRouter;

Copy
Copied!
Create a Next.js API route using the FileRouter
File path here doesn't matter, you can serve this from any route. We recommend serving it from /api/uploadthing.

pages/api/uploadthing.ts
import { createRouteHandler } from "uploadthing/next-legacy";
import { ourFileRouter } from "~/server/uploadthing";
export default createRouteHandler({
  router: ourFileRouter,
  // Apply an (optional) custom config:
  // config: { ... },
});

Copy
Copied!
See configuration options in server API reference

Create The UploadThing Components
src/utils/uploadthing.ts
import {
  generateUploadButton,
  generateUploadDropzone,
} from "@uploadthing/react";
import type { OurFileRouter } from "~/server/uploadthing";
export const UploadButton = generateUploadButton<OurFileRouter>();
export const UploadDropzone = generateUploadDropzone<OurFileRouter>();

Copy
Copied!
Add UploadThing's Styles
Tailwind
Not Tailwind
Wrap your Tailwind config with the withUt helper. You can learn more about our Tailwind helper in the "Theming" page

import { withUt } from "uploadthing/tw";
export default withUt({
  // Your existing Tailwind config
  content: ["./src/**/*.{ts,tsx,mdx}"],
  ...
});

Copy
Copied!
Mount A Button And Upload!
The @uploadthing/react package includes an "UploadButton" component that you can simply drop into your app, and start uploading files immediately.

app/example-uploader.tsx
import { UploadButton } from "~/utils/uploadthing";
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <UploadButton
        endpoint="imageUploader"
        onClientUploadComplete={(res) => {
          // Do something with the response
          console.log("Files: ", res);
          alert("Upload Completed");
        }}
        onUploadError={(error: Error) => {
          // Do something with the error.
          alert(`ERROR! ${error.message}`);
        }}
      />
    </main>
  );
}






Next.js App Router Setup
Oh, a little bleeding edge are we? We're big fans of app/ and server components, and we think you'll love what we've built 🙏

Setting up your environment
Install the packages
npm
pnpm
yarn
bun
npm install uploadthing @uploadthing/react

Copy
Copied!
Add env variables
.env
UPLOADTHING_TOKEN=... # A token for interacting with the SDK

Copy
Copied!
If you don't already have a uploadthing secret key, sign up ↗ and create one from the dashboard! ↗

Set Up A FileRouter
Creating your first FileRoute
All files uploaded to uploadthing are associated with a FileRoute. The following is a very minimalistic example, with a single FileRoute "imageUploader". Think of a FileRoute similar to an endpoint, it has:

Permitted types ["image", "video", etc]
Max file size
(Optional) middleware to authenticate and tag requests
onUploadComplete callback for when uploads are completed
To get full insight into what you can do with the FileRoutes, please refer to the File Router API.

app/api/uploadthing/core.ts
import { createUploadthing, type FileRouter } from "uploadthing/next";
import { UploadThingError } from "uploadthing/server";
const f = createUploadthing();
const auth = (req: Request) => ({ id: "fakeId" }); // Fake auth function
// FileRouter for your app, can contain multiple FileRoutes
export const ourFileRouter = {
  // Define as many FileRoutes as you like, each with a unique routeSlug
  imageUploader: f({ image: { maxFileSize: "4MB" } })
    // Set permissions and file types for this FileRoute
    .middleware(async ({ req }) => {
      // This code runs on your server before upload
      const user = await auth(req);
      // If you throw, the user will not be able to upload
      if (!user) throw new UploadThingError("Unauthorized");
      // Whatever is returned here is accessible in onUploadComplete as `metadata`
      return { userId: user.id };
    })
    .onUploadComplete(async ({ metadata, file }) => {
      // This code RUNS ON YOUR SERVER after upload
      console.log("Upload complete for userId:", metadata.userId);
      console.log("file url", file.url);
      // !!! Whatever is returned here is sent to the clientside `onClientUploadComplete` callback
      return { uploadedBy: metadata.userId };
    }),
} satisfies FileRouter;
export type OurFileRouter = typeof ourFileRouter;

Copy
Copied!
Create a Next.js API route using the FileRouter
File path here doesn't matter, you can serve this from any route. We recommend serving it from /api/uploadthing.

app/api/uploadthing/route.ts
import { createRouteHandler } from "uploadthing/next";
import { ourFileRouter } from "./core";
// Export routes for Next App Router
export const { GET, POST } = createRouteHandler({
  router: ourFileRouter,
  // Apply an (optional) custom config:
  // config: { ... },
});

Copy
Copied!
See configuration options in server API reference

Create The UploadThing Components
We provide components to make uploading easier. We highly recommend re-exporting them with the types assigned, but you CAN import the components individually from @uploadthing/react instead.

src/utils/uploadthing.ts
import {
  generateUploadButton,
  generateUploadDropzone,
} from "@uploadthing/react";
import type { OurFileRouter } from "~/app/api/uploadthing/core";
export const UploadButton = generateUploadButton<OurFileRouter>();
export const UploadDropzone = generateUploadDropzone<OurFileRouter>();

Copy
Copied!
Add UploadThing's Styles
Tailwind
Not Tailwind
Wrap your Tailwind config with the withUt helper. You can learn more about our Tailwind helper in the "Theming" page

import { withUt } from "uploadthing/tw";
export default withUt({
  // Your existing Tailwind config
  content: ["./src/**/*.{ts,tsx,mdx}"],
  ...
});

Copy
Copied!
Mount A Button And Upload!
Don't forget to add the "use client;" directive at the top of your file, since the UploadButton component needs to run on the client-side.

app/example-uploader/page.tsx
"use client";
import { UploadButton } from "~/utils/uploadthing";
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-between p-24">
      <UploadButton
        endpoint="imageUploader"
        onClientUploadComplete={(res) => {
          // Do something with the response
          console.log("Files: ", res);
          alert("Upload Completed");
        }}
        onUploadError={(error: Error) => {
          // Do something with the error.
          alert(`ERROR! ${error.message}`);
        }}
      />
    </main>
  );
}

Copy
Copied!
💡 Optional
Improving SSR
UploadThing needs to get info from your server to get permissions info. Normally this means a loading state. We built an optional plugin to prevent that

Without SSR Plugin
No file chosenReady
Allowed content
With SSR Plugin
No file chosenReady
Allowed content
To add SSR hydration and avoid that loading state, simply render the <NextSSRPlugin /> hydration helper in the body of your root layout before the children.

app/layout.tsx
import { NextSSRPlugin } from "@uploadthing/react/next-ssr-plugin";
import { extractRouterConfig } from "uploadthing/server";
import { ourFileRouter } from "~/app/api/uploadthing/core";
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <NextSSRPlugin
          /**
           * The `extractRouterConfig` will extract **only** the route configs
           * from the router to prevent additional information from being
           * leaked to the client. The data passed to the client is the same
           * as if you were to fetch `/api/uploadthing` directly.
           */
          routerConfig={extractRouterConfig(ourFileRouter)}
        />
        {children}
      </body>
    </html>
  );
}



