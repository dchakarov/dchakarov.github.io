---
title: My GitHub - A New Beginning
date: 2018-07-15 00:00:00 Z
layout: post
---

<span class="dropcap">A</span>fter I finished cleaning up [my GitHub account](https://github.com/dchakarov) last month it bugged me how empty it was. On the other side, I didn't want to upload just anything, as that would have lead to another clean up. So I spent a few days thinking about it and in the mean time I managed to read the book [App Architecture](https://www.objc.io/books/app-architecture/) (more on that in the next post) which inspired me to rewrite my two apps currently on the AppStore.

The first app - [Link As You Go](http://link-as-you-go.com) - I wrote [after a conference last year](https://medium.com/@gimly/using-cloud-ocr-to-parse-links-and-emails-from-photos-7815884a799) and although people praised me for it, it never got much traction. Which is sad, but on the bright side it means I can safely rewrite it without worrying about breaking it for anyone. Moreover, I could even open-source it and get some feedback from the community.

With that plan in mind, I started looking at the source code. I didn't want to publish it as is for several reasons, one of which was the API keys I had in the source code. So I decided to start with separating the Azure Computer Vision OCR service from the rest of the code and put it in a framework. It sounded like a good first candidate to be open-sourced.

Few hours later, I have the first _new_ repository on GitHub - [ComputerVisionOCR](https://github.com/dchakarov/ComputerVisionOCR). It is published under [The Unlicense](http://unlicense.org), which basically means you can do whatever, just don't blame me. As for the framework - it can help you send a photo with some text in it to the Azure servers and have the text extracted from it. The service is free for up to 5000 transactions (photos) a month and then you have to start paying. You can learn more about it [here](https://azure.microsoft.com/en-gb/pricing/details/cognitive-services/computer-vision/). My framework provides only the OCR service and for the moment is not configurable - the only thing you can change is the server URL you are using (they differ per continent/area).

If you like the framework and wanna help, jump in - PRs welcome!