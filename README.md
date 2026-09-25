# Kidonk Kidonk Web

Recreates the Kidonk Kidonk trend in a page styled like macOS Photos. You pick a photo, it fills 42 slots, and they get deleted one by one with a sound. Everything runs in the browser, and photos stay on the device in IndexedDB.

## Run locally

Any static server works. The camera input needs HTTPS or `localhost`.

```
npx serve .
```

## Deploy

Push to GitHub and import the repo in Vercel. Use framework preset "Other". There is no build command, and the output directory is the root.
