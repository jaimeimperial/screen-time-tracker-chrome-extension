author: Liam & Felix
summary: USF GDSC Chrome extension that keep track of the ammount of time that the user has spent on a website
id: usf-gdsc-chrome-extension
tags: workshop
categories: JavaScript
enviroments: web
status: published

# Screen Time Tracker

Chrome extensions enhance the browsing experience by customizing the user interface, observing browser events, and modifying the web. You can build extensions using the same web technologies that are used to create web applications: HTML, CSS, and JavaScript.

## Quick Introduction Crash Course on Extensions

**manifest.json:** This JSON file describes the extension's capabilities and configuration. For example, most manifest files contain an "action" key which declares the image Chrome should use as the extension's action icon and the HTML page to show in a popup when the extension's action icon is clicked. ONLY REQUIRED FILE.

**Service workers:** are special JavaScript environments that handle events and terminate when they're not needed. They allow extensions to monitor browser events in the background.

**Permissions:** are used to specify which resources and capabilities the extension needs to function properly. These permissions control what the extension is allowed to do and access on the user's behalf. 

**Icons:** play a significant role in representing your extension in the browser's toolbar, menus, and extension management pages. Can be specified in different sizes for various display contexts.

## Get Starter Code 

Head to the link below and fork the repo:

https://github.com/USFGDSC/screen-time-chrome-extension

Next clone the forked repository onto your machine onto your machine.

## Basic Background logic
Duration: 00:20:00

Create a **background.js file**. Add global variables that will be used throughout rest of program.

```
let currentTabId = null;
let startTime = Date.now();
```

Then create an alarm for tracking time in active tab every 5 seconds.

```
chrome.alarms.create('trackTime', { periodInMinutes: 1 / 12 }); // 5 seconds
```

Next we need to craete a funtion that handles the update current tab time functionality.

```
function updateCurrentTabTime() {
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    if (tabs.length === 0) return; // No active tab

    let tab = tabs[0];
    if (!tab.url) return; // No URL, likely a new tab or special page

    let url = new URL(tab.url).hostname;
    let currentTime = Date.now();

    // Check if we're still on the same tab
    if (currentTabId === tab.id) {
      let timeSpent = (currentTime - startTime) / 1000; // Calculate time spent in seconds
      updateSiteTime(url, timeSpent);
    }

    // Update the current tab and reset the start time
    currentTabId = tab.id;
    startTime = currentTime;
  });
}
```

The following step is to update the site time in storage.

```
function updateSiteTime(url, timeSpent) {
  chrome.storage.local.get({siteTime: {}}, function(data) {
    let siteTime = data.siteTime;
    if (siteTime[url]) {
      siteTime[url] += timeSpent;
    } else {
      siteTime[url] = timeSpent;
    }
    chrome.storage.local.set({siteTime}, () => {
      console.log('Site time updated', siteTime);
    });
  });
}
```

## Background Event Listeners
Duration: 00:15:00

First, we need to incorporate event listener that listens for the alarm and then update the time spent on the current site.

```
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'trackTime') {
    updateCurrentTabTime();
  }
});
```

Next, we need to listen for tab activation to handle tab switching.

```
chrome.tabs.onActivated.addListener(function(activeInfo) {
  // Reset start time when switching tabs
  startTime = Date.now();
  currentTabId = activeInfo.tabId;
});
```

Then, we also need to listen for tab updates to catch navigations in the same tab.

```
chrome.tabs.onUpdated.addListener(function(tabId, changeInfo, tab) {
  if (tab.active && changeInfo.url) {
    let url = new URL(changeInfo.url).hostname;
    let currentTime = Date.now();
    let timeSpent = (currentTime - startTime) / 1000;
    updateSiteTime(url, timeSpent);
    // Reset the timer for the current tab
    startTime = currentTime;
    currentTabId = tabId;
  }
});
```

Finally, we need to incorporate listening for window focus changes to reset the timer appropriately.

```
chrome.windows.onFocusChanged.addListener(function(windowId) {
  if (windowId === chrome.windows.WINDOW_ID_NONE) {
    currentTabId = null; // No window is focused
  } else {
    chrome.tabs.query({active: true, windowId: windowId}, function(tabs) {
      if (tabs.length > 0) {
        currentTabId = tabs[0].id;
      }
    });
  }
  startTime = Date.now(); // Reset start time whenever window focus changes
});
```

## Load Extension

To load an unpacked extension in developer mode:

1. Go to the Extensions page by entering chrome://extensions in a new tab. (By design chrome:// URLs are not linkable.)

  Alternatively, click the Extensions menu puzzle button and select Manage Extensions at the bottom of the menu.

  Or, click the Chrome menu, hover over More Tools, then select Extensions.

2. Enable Developer Mode by clicking the toggle switch next to Developer mode.

3. Click the Load unpacked button and select the extension directory.

Ta-da! The extension has been successfully installed. 

Now pin it and try it out. If anything does not work as intended check Developer tools for erros and check/modify code. If modidying anything change the name of the extension in **manifest.json** and click the reload icon on the Extensions Page.

## Add Changes to Repository

Run following terminal commands to upload complete code to your GitHub:

```
git add .
```

```
git commit -m "finished"
```

## CONGRATULATIONS! YOU HAVE A FINISHED YOUR EXTENSION!
