# #dev-talk: our implementation of lazy loading responsive images with the image-observer and picture element
With the, still, growing use of mobile devices,  the relevance of fast, responsive website is growing also. In this blogpost I show you an example of how we use images-lazy-loading and the html5-picture element to improve the performance of our images on all devices. I do not go very deep into the technical details. To read more about all the IntersectionObserver and a full implementation-guide; I refer to the links at the bottom of this article.

## Lazy-loading
The average user of your websites does not see all the images on a page. Because he does not scroll far enough or immediately clicks through to another page. Loading all images on a page beforehand is therefore often unnecessary. Fortunately we can prevent this with the help of images-lazy-loading. We will not load the images until the images is almost in the viewport of the user. We use a new browser-feature for this: the [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API). In this blogpost I will show you how we implemented this.


## hoe werkt image-lazy-loading
With the help of the [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) we can recognize if an certain html-element is in the viewport. If this is the case, we will load the images. 

``` Html
<picture>
    <!-- Data-srcset and data-src does nothing but is needed for lazy-loading. If the image is in the viewport, the data-srcset/data-src is replaced by the attribute srcset/src. When this is the case the image is loaded. -->
    <source media="(max-width: 768px)" data-srcset="image-768x768.jpg, image-1536x1536.jpg 2x">
    <source media="(min-width: 768px)" data-srcset="image-1536x1536.jpg, image-2400x2400.jpg 2x">
    <!-- load low quality-blurry image by default -->
    <img src="image-15x15.jpg" data-src="image-768x768.jpg">
</picture>
```

``` JavaScript

// select both source and image elements -> more on this later
const images = window.document.querySelectorAll('source, img');

// Some config parameters for the IntersectionObserver
const config = {
    // If the image gets within 50px in the Y axis, start the download.
    rootMargin: '50px 0px',
    threshold: 0.01
};

let observer;

// If we don't have support for intersection observer, load the images immediately
if (!('IntersectionObserver' in window)) {
    Array.from(images).forEach(image => preloadImage(image));
}
else {
    // It is supported, load the images by calling our method: onIntersection
    observer = new IntersectionObserver(onIntersection, config);

    images.forEach(image => {
        observer.observe(image);
    });
}


// Replace the data-src attribute with the value of the data-src attribute
let preloadImage = (element) => {
    if(element.dataset && element.dataset.src) {
        element.src = element.dataset.src;
    }
    if(element.dataset && element.dataset.srcset) {
        element.srcset = element.dataset.srcset
    }
}

let onIntersection = (entries) => {
    entries.forEach(entry => {
        if (entry.intersectionRatio > 0) {

            // Stop watching and load the image
            observer.unobserve(entry.target);
            // call our method: preloadImage
            preloadImage(entry.target);
        }
    });
}

```

[The picture element(1)](https://caniuse.com/#search=picture)
[The picture element(2)](https://developer.mozilla.org/nl/docs/Web/HTML/Element/picture)
[More about lazy loading images](http://deanhume.com/home/blogpost/lazy-loading-images-using-intersection-observer/10163)
