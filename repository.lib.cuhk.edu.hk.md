# repository.lib.cuhk.edu.hk

## 1. Intro
This site uses a framework called `islandora`. You can find already-created scripts for cloning media hosted on this framework, but here I will go on my own (yes it's stupid).

## 2. Get all image IDs
Example: cloning all images from https://repository.lib.cuhk.edu.hk/en/item/cuhk-1941296

Switch to <kbd>Pages</kbd> tab and click on <kbd>List view</kbd>. You will get to [this]([https://repository.lib.cuhk.edu.hk/en/islandora/object/cuhk%3A1941296/pages](https://repository.lib.cuhk.edu.hk/en/islandora/object/cuhk%3A1941296/pages?display=list)) page and see this:
![image](https://github.com/user-attachments/assets/4a119219-2f88-482c-adf3-b184c258be32)

Open Devtool using <kbd>F12</kbd>, go to <kbd>Console</kbd> tab and paste this code in:
```js
// Get the container with the list of items
const container = document.querySelector("#content > div > div > div.islandora-objects.clearfix > div.islandora-objects-list");

// Select all `.islandora-objects-list-item` elements within the container
const imageItems = container.querySelectorAll(".islandora-objects-list-item");

// Use a Set to avoid duplicate IDs
const idsSet = new Set();

// Loop through each image item and extract the full ID from the first anchor tag
imageItems.forEach(item => {
    const link = item.querySelector('a'); // Select the first <a> element within the item

    if (link) {
        const href = link.getAttribute("href"); // Get the href attribute of the anchor tag
        
        // Extract the full ID, including the `3A` part
        const segments = href.split('/');
        const fullId = segments[segments.length - 1]; // Get the last part of the URL

        // Make sure to include IDs with `3A` or `:`
        if (fullId.includes('%3A') || fullId.includes(':')) {
            idsSet.add(fullId); // Add the found ID to the set
        }
    }
});

// Convert the Set to an array
const newIds = Array.from(idsSet);

// Retrieve the existing array from localStorage
const existingIdsJson = localStorage.getItem('imageIds');
let existingIds = [];

// If existing data is found, parse it to an array
if (existingIdsJson) {
    existingIds = JSON.parse(existingIdsJson);
}

// Concatenate new IDs, dedupe, and save back to localStorage
const allIds = Array.from(new Set(existingIds.concat(newIds)));
localStorage.setItem('imageIds', JSON.stringify(allIds));
```

Press enter. When done, click <kbd>next</kbd> on the page to do the same with other pages. After you have cycled over all the pages, go to <kbd>Application</kbd> tab, and browse for the `imageIds` key in the local storage:
![image](https://github.com/user-attachments/assets/e6a4e0fd-b910-4281-bd43-54f4bba90eda)

Copy the key's value and then **DELETE IT** (If you don't delete it, your next book will mess with your current book. Yes you can do it if that's really what you want).

Example value:
```python
["cuhk%3A1941318","cuhk%3A1941341","cuhk%3A1941340","cuhk%3A1941358","cuhk%3A1941403","cuhk%3A1941412","cuhk%3A1941308","cuhk%3A1941303","cuhk%3A1941359","cuhk%3A1941309","cuhk%3A1941327","cuhk%3A1941418"]
```

## 3. First steps at Python prompt
Install Python 3.x and curl.

Create a folder on your computer, and open Python REPL.

Write `a = [YOUR,VALUE,HERE]` (replace `[YOUR,VALUE,HERE]` with your newly-copied book IDs array) and then press enter. For example, write `a = ["cuhk%3A1941318","cuhk%3A1941341","cuhk%3A1941340","cuhk%3A1941358","cuhk%3A1941403","cuhk%3A1941412","cuhk%3A1941308","cuhk%3A1941303","cuhk%3A1941359","cuhk%3A1941309","cuhk%3A1941327","cuhk%3A1941418"]` then press enter.

## 4. Get cookie
Next, get cookie from your page by going to <kbd>Network</kbd> tab, clicking on <kbd>Doc</kbd> filter, and selecting the first one. Cookie field should be visible. Copy that.
![image](https://github.com/user-attachments/assets/46bb0b8e-d316-4841-beec-2f6a1b80b162)

## 5. Going back to Python prompt
Copy this code to a text editor (word wrap on recommended):
```python
import os
cookie = '' # Cookie string here
s = 0       # Start/resume image number here
c = s
for i in a[s:]:
    os.system('curl "https://repository.lib.cuhk.edu.hk/iiif/2/' + i + '~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Host: repository.lib.cuhk.edu.hk" -H "Connection: keep-alive" -H "Cache-Control: max-age=0" -H "sec-ch-ua: \"Google Chrome\";v=\"131\", \"Chromium\";v=\"131\", \"Not_A Brand\";v=\"24\"" -H "sec-ch-ua-mobile: ?0" -H "sec-ch-ua-platform: \"Windows\"" -H "Upgrade-Insecure-Requests: 1" -H "DNT: 1" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36" -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7" -H "Sec-Fetch-Site: same-origin" -H "Sec-Fetch-Mode: navigate" -H "Sec-Fetch-Dest: document" -H "Referer: https://repository.lib.cuhk.edu.hk/iiif/2/cuhk%%3A1940967~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Accept-Encoding: gzip, deflate, br, zstd" -H "Accept-Language: en,vi;q=0.9,en-US;q=0.8,zh-CN;q=0.7,zh;q=0.6" -H "Cookie: ' + cookie + '" -o ' + str(c).zfill(4) + ".jpg")
    c += 1
```

Modify cookie field: paste your cookie string in. Example:
```python
import os
cookie = 'textsize=100; SSESSa2a27be57fb952eee82d64cf4870c511=VRhQOpPtheCmSmONMvPUJ49d_oJSM_nl1_OZXnBlPko; SESSa2a27be57fb952eee82d64cf4870c511=c387MymdsEwYyoa-dCBykl3OiNKBVpYDhdnW7hMAQUg; SL_G_WPT_TO=en; SL_GWPT_Show_Hide_tmp=1; SL_wptGlobTipTmp=1; aws-waf-token=089fc639-a03a-4b75-8abb-0b33bfb1df79:BgoAfx5IT/w6AQAA:8Imvg3+3dS6OgGJgQtyBTM2uB8I/xcUQQN+3iW0WsaHZmVem5vFl6og0aQyz+0jDX/cQvHKEwiAg9tmvW/jIKMQgSbivwO6XjVGLSbIO4o56LJs4eGy6y6q48w7LMbUqn9HUZ3ov56AP+jNgXITB6nLd04yTpTxPyoMlh86FCxefrefTIn4TX5LRDrSqsZTifXEHFFhueSKzSSkcgcjeugW2xvNNULVaFsEeyX1UJbWqq/j8wTpdNdyYxdV0ZLX2cQN2iS+bbxcACu+E59tr06RPgJg='
s = 0
c = s
for i in a[s:]:
    os.system('curl "https://repository.lib.cuhk.edu.hk/iiif/2/' + i + '~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Host: repository.lib.cuhk.edu.hk" -H "Connection: keep-alive" -H "Cache-Control: max-age=0" -H "sec-ch-ua: \"Google Chrome\";v=\"131\", \"Chromium\";v=\"131\", \"Not_A Brand\";v=\"24\"" -H "sec-ch-ua-mobile: ?0" -H "sec-ch-ua-platform: \"Windows\"" -H "Upgrade-Insecure-Requests: 1" -H "DNT: 1" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36" -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7" -H "Sec-Fetch-Site: same-origin" -H "Sec-Fetch-Mode: navigate" -H "Sec-Fetch-Dest: document" -H "Referer: https://repository.lib.cuhk.edu.hk/iiif/2/cuhk%%3A1940967~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Accept-Encoding: gzip, deflate, br, zstd" -H "Accept-Language: en,vi;q=0.9,en-US;q=0.8,zh-CN;q=0.7,zh;q=0.6" -H "Cookie: ' + cookie + '" -o ' + str(c).zfill(4) + ".jpg")
    c += 1
```

Press enter. Your download should start.

## 6. MUST READ: Help! My file is empty.
Sometimes you get some 2-3KBs -ish files that are not images. Delete these files. Note the file with **biggest** number remained. Let call that number `n` (for example, n = 50).

Then, go back to step 4. Refresh your page by <kbd>F5</kbd> and then get a new cookie string.

Next, go to step 5 and put the new cookie string in. The `s` variable in the script now should be the number you noted before **PLUS 1**. For example, if `n = 50`, put `s = 51` in. Example script:
```python
import os
cookie = 'textsize=100; SSESSa2a27be57fb952eee82d64cf4870c511=VRhQOpPtheCmSmONMvPUJ49d_oJSM_nl1_OZXnBlPko; SESSa2a27be57fb952eee82d64cf4870c511=c387MymdsEwYyoa-dCBykl3OiNKBVpYDhdnW7hMAQUg; SL_G_WPT_TO=en; SL_GWPT_Show_Hide_tmp=1; SL_wptGlobTipTmp=1; aws-waf-token=089fc639-a03a-4b75-8abb-0b33bfb1df79:BgoAfx5IT/w6AQAA:8Imvg3+3dS6OgGJgQtyBTM2uB8I/xcUQQN+3iW0WsaHZmVem5vFl6og0aQyz+0jDX/cQvHKEwiAg9tmvW/jIKMQgSbivwO6XjVGLSbIO4o56LJs4eGy6y6q48w7LMbUqn9HUZ3ov56AP+jNgXITB6nLd04yTpTxPyoMlh86FCxefrefTIn4TX5LRDrSqsZTifXEHFFhueSKzSSkcgcjeugW2xvNNULVaFsEeyX1UJbWqq/j8wTpdNdyYxdV0ZLX2cQN2iS+bbxcACu+E59tr06RPgJg='
s = 51
c = s
for i in a[s:]:
    os.system('curl "https://repository.lib.cuhk.edu.hk/iiif/2/' + i + '~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Host: repository.lib.cuhk.edu.hk" -H "Connection: keep-alive" -H "Cache-Control: max-age=0" -H "sec-ch-ua: \"Google Chrome\";v=\"131\", \"Chromium\";v=\"131\", \"Not_A Brand\";v=\"24\"" -H "sec-ch-ua-mobile: ?0" -H "sec-ch-ua-platform: \"Windows\"" -H "Upgrade-Insecure-Requests: 1" -H "DNT: 1" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36" -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7" -H "Sec-Fetch-Site: same-origin" -H "Sec-Fetch-Mode: navigate" -H "Sec-Fetch-Dest: document" -H "Referer: https://repository.lib.cuhk.edu.hk/iiif/2/cuhk%%3A1940967~JP2~public_default~d37828a6613b15e131d3c9a1bfb02e127d4959dd58dd678a55cbb43ddfdcf815/full/pct:100/0/default.jpg" -H "Accept-Encoding: gzip, deflate, br, zstd" -H "Accept-Language: en,vi;q=0.9,en-US;q=0.8,zh-CN;q=0.7,zh;q=0.6" -H "Cookie: ' + cookie + '" -o ' + str(c).zfill(4) + ".jpg")
    c += 1
```

Press enter again. Your download should now resume. Repeat instructions in step 6 whenever this issue happends.
