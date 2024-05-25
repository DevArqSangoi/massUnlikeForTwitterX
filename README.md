# Auto Unlike and Scroll

This Tampermonkey script is designed to automate the process of clicking "unlike" buttons on TwitterX and subsequently scrolling down the page.

## How It Works

The script operates by scanning the page for elements marked with a `data-testid="unlike"`. Upon finding these elements, it performs the following actions:
1. Clicks on each "unlike" button with a random delay between each click.
2. Waits for a couple of seconds after each click to ensure that the page's state has updated to "like".
3. After attempting to unlike all detected items, it rechecks to confirm all have been processed. If no "unlikes" remain, the script triggers a page scroll.

## Installation

To use this script, follow these steps:
1. Install the Tampermonkey (or Greasemonkey) extension available for Chrome, Firefox, Safari, or any other compatible browser from [Tampermonkey's official website](https://www.tampermonkey.net/).
2. Open the Tampermonkey dashboard and click on "Create a new script".
3. Copy and paste the following script into the script editor:

## Usage

To use this script effectively, simply follow these steps:
1. Navigate to your likes tab and wait. The script may take a few seconds to start, but it will start. The URL should follow this pattern: `https://x.com/{your_username}/likes`.
2. Once you are on the correct page, the script will automatically start to identify and click any "unlike" buttons present on the page.
3. The script operates in intervals, attempting to click "unlikes", verifying changes, and then scrolling down to load more items if no more "unlikes" are detected.
4. Monitor the process, if necessary, to ensure that everything is functioning as expected. Console logs will provide real-time feedback on the script’s actions, indicating which elements are being clicked and the status of the script’s operations.
Remember, the script will continuously run as long as you are on the likes page, managing the unlikes and scrolling automatically to uncover more items if needed. Ensure that Tampermonkey is enabled and the script is active before visiting the page.


```javascript
// ==UserScript==
// @name         Auto Unlike and Scroll
// @namespace    https://github.com/DevArqSangoi
// @version      0.1
// @description  Automatically clicks on 'unlikes' and scrolls the page
// @author       You
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    let isProcessing = false; // Flag to prevent multiple invocations
    let lastInvocationTime = Date.now(); // Track last invocation time for cooldown

    function clickElement(element) {
        const clickEvent = new MouseEvent('click', {
            'view': window,
            'bubbles': true,
            'cancelable': false
        });
        element.dispatchEvent(clickEvent);
        console.log('Clicked on element.');
    }

    async function verifyAndClick(hearts) {
        for (const heart of hearts) {
            clickElement(heart);
            await new Promise(resolve => setTimeout(resolve, 2000)); // Wait to see if state changes
            if (heart.getAttribute('data-testid') !== 'like') {
                console.log('State is still unlike, trying to click again.');
                clickElement(heart);
            }
        }
        checkCompletion();
    }

    function checkCompletion() {
        let checkHearts = document.querySelectorAll('button[data-testid="unlike"]');
        if (checkHearts.length === 0) {
            console.log('No unlikes left, scrolling');
            scrollToBottom();
        } else {
            console.log('Unlikes still exist, will not scroll');
        }
        isProcessing = false; // Reset flag after processing
    }

    function scrollToBottom() {
        window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
    }

    function processHearts() {
        let now = Date.now();
        if (!isProcessing && (now - lastInvocationTime > 5000)) { // Ensure there's a 5-second cooldown
            isProcessing = true; // Set flag to true to indicate processing is ongoing
            lastInvocationTime = now; // Update last invocation time
            let hearts = document.querySelectorAll('button[data-testid="unlike"]');
            console.log('Hearts found:', hearts.length);
            if (hearts.length > 0) {
                verifyAndClick(Array.from(hearts));
            } else {
                console.log('No unlikes found, performing scroll');
                scrollToBottom();
                isProcessing = false; // Reset flag if nothing to process
            }
        }
    }

    // Observe DOM changes for new unlikes being added dynamically
    function observeDOM() {
        const observer = new MutationObserver((mutations) => {
            // Check if the current URL matches the intended pattern
            if (/^https?:\/\/x\.com\/[^/]*\/likes/.test(window.location.href)) {
                mutations.forEach((mutation) => {
                    if (mutation.addedNodes.length) {
                        processHearts();
                    }
                });
            }
        });

        observer.observe(document.body, {
            childList: true,
            subtree: true
        });
    }

    // Initialize the observer and start processing
    observeDOM();
})();
