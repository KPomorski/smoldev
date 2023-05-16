a Chrome Manifest V3 extension that reads the current page, and offers a popup UI that has the page title+content and a textarea for an ingredient list (with the default ingredients list coming from the content of the page). When the user hits submit, it saves the up-to-date ingredients list to a JSON-formatted file. The user can then choose to download that file.

- Only when clicked:
  - it injects a content script `content_script.js` on the currently open tab, and accesses the title `pageTitle` and main content (innerText) `pageContent` of the currently open page 
  (extracted via an injected content script, and sent over using a `storePageContent` action) 
  - in the background, receives the `storePageContent` data and stores it
  - only once the new page content is stored, then it pops up a full height window with a minimalistic styled html popup
  - in the popup script
    - the popup should display a 10px tall rounded css animated red and white candy stripe loading indicator `loadingIndicator`, while waiting the JSON file is created
      - with the currently fetching page title and a running timer in the center showing time elapsed since the process started 
      - hide the running timer when it ends.
    - retrieves the page content data using a `getPageContent` action (and the background listens for the `getPageContent` action and retrieves that data) and displays the title at the top of the popup
    - on the bottom of the popup, there is a nicely styled submit button with an id of `sendButton` (tactile styling that "depresses" on click)
      - only when `sendButton` is clicked, we create the JSON-formatted file, which is saved as the title of the recipe .json

Important Details:

- It has to run in a browser environment, so no Nodejs APIs allowed.

- The webpage contents will need to be scraped using some type of scraping method to validate whether there is a recipe, and what the ingredients are.

- add styles to make sure the popup's styling follows the basic rules of web design, for example having margins around the body, and a system font stack.

- style the popup body with <link rel="stylesheet" href="https://unpkg.com/mvp.css@1.12/mvp.css"> but insist on body margins of 16 and a minimum width of 400 and height of 600.

## debugging notes

inside of background.js, just take the getPageContent response directly

```js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'storePageContent') {
    // dont access request.pageContent
    chrome.storage.local.set({ pageContent: request }, () => {
      sendResponse({ success: true });
    });
  } else if (request.action === 'getPageContent') {
    chrome.storage.local.get(['pageContent'], (result) => {
      // dont access request.pageContent
      sendResponse(result);
    });
  }
  return true;
});
```
