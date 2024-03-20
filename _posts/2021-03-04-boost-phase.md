---
title: "Drowning in Boost"
categories: 
    - C++
tags: 
    - Boost GIL
    - C++
    - Bicubic sampler
layouts: single
classes: wide
sidebar:
    nav : gsoc
gallery:
    - url: /assets/images/toy_car.png
      image_path: /assets/images/toy_car.png
      alt: Toy Cars
      title: Original Image
    - url: /assets/images/toy_car_scaled.png
      image_path: /assets/images/toy_car_scaled.png
      alt: Toy Cars Scaled with artefacts
      title: Scaled Image
---

<img src="{{ site.baseurl }}/assets/images/boost.jpeg" width=500>  

Yes, I tried contributing to the boost c++ Generic Image library(GIL). I really wanted to learn how production C++ code is like. And boy did I find out the hard way.  

So, the thing is - C++ is not the best language for a new programmer to jump into, especially without reading any books for reference, since there are minimal resources online and I also did not know how to search for stuff as I didn't know the keywords. All of this lead to me spending hours looking at the code of the GIL library and figuring out through trial and error and through the mailing lists (I did not even think of searching for C++ resources 😭).  

Moreover there was I thing I did not realize, it is better to have a basic idea of some parts and proceed with that, then learn along the way instead of waiting to understand the whole codebase to start work.

After all this I had a basic idea of how some parts of it worked, (I gave up on understanding all of it, thankfully) and decided to implement a bicubic sampler for resizing an image in the GIL library. I was going to implement this as an example so later if it was a good implementation, it could be made into a part of the library.  

Since this was also around the time of GSOC, I thought I could also try for that since why not. I raised an issue in the GIL repository and started implementing it. Only after the GSOC candidates were selected did I realize that one should mail the maintainers to check with them and inform them of your intentions, before submitting the GSOC application, otherwise the chances of getting selected are low.  

Anyways I tried implementing the bicubic sampler. In the first iteration, it had a weird issue where there were artefacts in the scaled image, it looked like there was an overflow in some calculation. 

![Original Image]({{ site.baseurl }}/assets/images/toy_car.png)  
Original Image  
<img src="{{ site.baseurl }}/assets/images/toy_car_scaled.png" width=600>  
Scaled Image  

Now since I had not faced such issues with my code before, I didn't know how to find its origin. So I posted my findings in the github issue I had raised, and I got nowhere.  
The only solution I could see was by limiting the values the pixels could take, making the operation reset to the max value when it overflows, but I didn't find it elegant.  

Though, I couldn't find out how the overflow was happening and had to resort to this solution after some days passed. I raised a pr for this solution, [where it still exists](https://github.com/boostorg/gil/pull/588)...