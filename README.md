# hexo-photoswipe

## What is hexo-photoswipe

When you use hexo to build an vanilla blog, you might want a fine gallery to exhibit photos you uploaded. 

`Hexo-photoswipe` is the one that would power you and your photos up.

## How to use

Note that, this is not an **Out of the box** plugin, because it's built for me personally. I'll be thrilled if it inspired you some kind.

Note that, `hexo-photoswipe` is based on [hexo-lazyload-image](https://www.npmjs.com/package/hexo-lazyload-image), which will generate the `data-original` attr in `img` tag.

Note that, this plugin detect the image size/resolution two ways.

One is that images are stored in `yourTitle/someImage.format` which means images are introduced in {% asset_img foo.bar "foobar some text" %} way.

The other is that images are quoted as http(s), which means images are introduced in ![foobar some text](http(s)://www.john.doe/foo.bar) way

### install 

So, first you install `hexo-lazyload-image`:

```shell
npm install hexo-lazyload-image --save
```

or my version, which killed some annoying console.log(...)

```shell
npm install hexo-lazyload-image-modified --save
```

Additionally, install `hexo-photoswipe`:

```shell
npm install hexo-photoswipe --save
```

and boom, you have a `div`-wrapped `img` tag which also contains something like `class="image-container" data-type="content-image" data-size=100x100` in you final render result.

Finally, you write some code using [photoswipe](https://photoswipe.com/) in your `somefilename.js`

> If you have no idea what to do, please open an issue at github: <https://github.com/HarborZeng/hexo-photoswipe/issues>

## demo

![demo](https://github.com/HarborZeng/hexo-photoswipe/blob/master/presentation.gif)

## LICENSE

MIT

## Demo

Suppose you have installed the dependencies above. Then you may proceed.

### 0. Enable the plugin

in your hexo blog's `_config.yml`:

```yaml
# <div class="image-container" data-type="content-image" data-size="100x100"><img src="xxx" data-original="yyy"></img></div>'
photoswipe:
  enable: true
  className: image-container
  dataType: content-image
```

### 1. include JS and CSS files

You can find them in [dist/](https://github.com/dimsemenov/PhotoSwipe/tree/master/dist) folder of [GitHub](https://github.com/dimsemenov/PhotoSwipe) repository. Sass and uncompiled JS files are in folder [src/](https://github.com/dimsemenov/PhotoSwipe/tree/master/src).

It doesn't matter how and where will you include JS and CSS files. Code is executed only when you call new PhotoSwipe(). So feel free to defer loading of files if you don't need PhotoSwipe to be opened initially.

or

It's highly recommended to use cdn

```html
<script src="https://cdn.bootcss.com/photoswipe/4.1.3/photoswipe.min.js"></script>
<script src="https://cdn.bootcss.com/photoswipe/4.1.3/photoswipe-ui-default.min.js"></script>

<link href="https://cdn.bootcss.com/photoswipe/4.1.3/photoswipe.min.css" rel="stylesheet">
<link href="https://cdn.bootcss.com/photoswipe/4.1.3/default-skin/default-skin.min.css" rel="stylesheet">
```

### 2. add `pswp` codes into your layout page

you can add the code below to your theme's `main page` or `layout.ejs` code

This code can be appended anywhere, but ideally before the closing </body>. You may reuse it across multiple galleries (as long as you use same UI class).

```html
<!-- Root element of PhotoSwipe. Must have class pswp. -->
<div class="pswp" tabindex="-1" role="dialog" aria-hidden="true">

    <!-- Background of PhotoSwipe.
         It's a separate element as animating opacity is faster than rgba(). -->
    <div class="pswp__bg"></div>

    <!-- Slides wrapper with overflow:hidden. -->
    <div class="pswp__scroll-wrap">

        <!-- Container that holds slides.
            PhotoSwipe keeps only 3 of them in the DOM to save memory.
            Don't modify these 3 pswp__item elements, data is added later on. -->
        <div class="pswp__container">
            <div class="pswp__item"></div>
            <div class="pswp__item"></div>
            <div class="pswp__item"></div>
        </div>

        <!-- Default (PhotoSwipeUI_Default) interface on top of sliding area. Can be changed. -->
        <div class="pswp__ui pswp__ui--hidden">

            <div class="pswp__top-bar">

                <!--  Controls are self-explanatory. Order can be changed. -->

                <div class="pswp__counter"></div>

                <button class="pswp__button pswp__button--close" title="Close (Esc)"></button>

                <button class="pswp__button pswp__button--share" title="Share"></button>

                <button class="pswp__button pswp__button--fs" title="Toggle fullscreen"></button>

                <button class="pswp__button pswp__button--zoom" title="Zoom in/out"></button>

                <div class="pswp__preloader">
                    <div class="pswp__preloader__icn">
                        <div class="pswp__preloader__cut">
                            <div class="pswp__preloader__donut"></div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="pswp__share-modal pswp__share-modal--hidden pswp__single-tap">
                <div class="pswp__share-tooltip"></div>
            </div>

            <button class="pswp__button pswp__button--arrow--left" title="Previous (arrow left)">
            </button>

            <button class="pswp__button pswp__button--arrow--right" title="Next (arrow right)">
            </button>

            <div class="pswp__caption">
                <div class="pswp__caption__center"></div>
            </div>

        </div>

    </div>

</div>
```

### 3. Initialize in your own js file

```javascript
let pswpElement = document.querySelector('.pswp')
let imgSrcItem = []
$('someSelectorToYourImage').each(function (index) {
  // data-original is generated by hexo-lazyload-image,
  // which represent the src of the image
  let imgPath = this.getAttribute('data-original')
  
  $(this).addClass("some class if you want")
  
  // caption text
  let alt = $(this).attr('alt')
  let title = $(this).attr('title')
  let captionText = alt || title
  if (captionText) {
    let captionDiv = document.createElement('div')
    captionDiv.className += ' caption'
    let captionEle = document.createElement('b')
    captionEle.className += ' center-caption'
    captionEle.innerText = captionText
    captionDiv.appendChild(captionEle)
    
    // insert where appropriate
    this.parentElement.insertAdjacentElement('afterend', captionDiv)
  }
  
  if (this.parentNode.getAttribute('data-size')) {
    // images introduced in {% asset_img foo.bar "foobar some text" %} way
    let resolution = this.parentNode.getAttribute('data-size').split('x')
    imgSrcItem.push({
      src: imgPath,
      w: resolution[0],
      h: resolution[1],
      title: captionText
    })
  } else {
    // images introduced in ![foobar some text](http(s)://www.john.doe/foo.bar) way
    imgSrcItem.push({
        src: imgPath,
        w: this.naturalWidth || window.innerWidth,
        h: this.naturalHeight || window.innerHeight,
        title: captionText
    })
    
    // when the image loaded, update the w and h
    this.addEventListener('load', function (e) {
        imgSrcItem[index].w = this.naturalWidth
        imgSrcItem[index].h = this.naturalHeight
    })
  }
})

$('#articleContent img').each(function (i) {
  $(this).click(function (e) {
    let options = {
      // https://photoswipe.com/documentation/options.html
      // optionName: 'option value'
      // for example:
      index: i, // start at first slide
      barsSize: {top: 0, bottom: 0},
      captionEl: true,
      fullscreenEl: false,
      shareEl: false,
      bgOpacity: 0.5,
      tapToClose: true,
      tapToToggleControls: true,
      showHideOpacity: false,
      counterEl: true,
      preloaderEl: true,
      history: false,
    }
    let gallery = new PhotoSwipe(pswpElement, PhotoSwipeUI_Default, imgSrcItem, options)
    gallery.listen('imageLoadComplete', function (index, item) {
      // index - index of a slide that was loaded
      // item - slide object
      // update src of images in web page when their view are not reached by browser,
      // while photoswipe has completely load the image 
      $('#articleContent img')[index].src = item.src
    })
    gallery.init()
  })
})
```

### 4. Add css into your project (optional)

```css
.image-container {
  text-align: center;
  /*...more style...*/
}
```
