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
1. Navigate to the likes tab of the profile whose likes you want to manage. The URL should follow this pattern: `https://x.com/{your_username}/likes`.
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
// @match        *://x.com/*/likes
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    function clickElement(element) {
        const clickEvent = new MouseEvent('click', {
            'view': window,
            'bubbles': true,
            'cancelable': false
        });
        element.dispatchEvent(clickEvent);
        console.log('Clicked on element.');
    }

    function verifyChange(element, index, total) {
        setTimeout(() => {
            if (element.getAttribute('data-testid') === 'like') {
                console.log('State changed to like.');
                if (index === total - 1) {
                    setTimeout(checkCompletion, 3000); // Delay before checking completion
                }
            } else {
                console.log('State is still unlike, trying to click again.');
                clickElement(element);
                setTimeout(() => verifyChange(element, index, total), 2000); // Reschedule the check if the state has not changed
            }
        }, 2000); // Wait 2 seconds to check if the state has changed
    }

    function checkCompletion() {
        let checkHearts = Array.from(document.querySelectorAll('button[data-testid="unlike"]'));
        console.log('Rechecking hearts after clicks.');
        if (checkHearts.length === 0) {
            console.log('Nenhum unlike restante, fazendo scroll');
            window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
        } else {
            console.log('Unlikes still exist, will not scroll');
        }
    }

    function processHearts() {
        let buttons = document.querySelectorAll('button[data-testid="unlike"]');
        let hearts = Array.from(buttons);

        console.log('Hearts found:', hearts.length);

        if (hearts.length > 0) {
            hearts.forEach((h, index) => {
                setTimeout(() => {
                    clickElement(h);
                    verifyChange(h, index, hearts.length);
                }, index * 2000); // Delay between each click
            });
        } else {
            console.log('No unlikes found, performing scroll');
            window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
        }
    }

    setInterval(processHearts, 5000 + Math.random() * 1000); // Interval to avoid overly rapid actions
})();
