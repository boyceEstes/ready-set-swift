# SF Symbols

Including in your project

```
Image(systemName: "bolt")
```

This is all you need to get started. There are lots of things you can do further to get it exactly how you want it, however.

To give it a set size you can treat this like any other image and apply the following modifiers however it suits ya

```
Image(systemName: "bolt")
	.resizable()
	.aspectRatio(.fit) // Fill if you want it to possibly exceed the frame
	.frame(width: 24)
``` 


## Dynamic resizing
Instead of resizing with a static, unchanging value, you can set it up to respond to dynamic type in a couple of different ways.


### TLDR
Below you can read some details about how you can set these system images to be dynamically resized using `.font` OR `.imageScale`. 

In my experimentation, the `.font(_:)` method never reliably worked. That being said. `.imageScale(_:)` will probably be your best bet.

Using `.font(.system(size:weight:))` would get you the MOST customization options (but will probably resize dynamically).

**Note**: Beware of putting frames on the image that you want to resize dynamically as this can cause problems if you do not set the aspectRatio and it exceeds the frame.

### Font
By treating the image like a `Text` View and setting a font you can resize a symbol correlating to some font size. This is actually a very unreliable method to use for dynamic type conformation. Use [Image scale](#image-scale) instead.

```
Image(systemName: "bolt")
	.font(.system(size: 20))
```

You can similarly set weight like a normal font to make the symbol more or less bold.

```
Image(systemName: "bolt")
	.font(.system(size: 20, weight: .semibold))
```


### Image scale
The imageScale modifier can also be used on SF Symbols to give a rough approximation on the size of your image. 

```
Image(systemName: "bolt")
	.imageScale(.medium)
```

The three available sizes are: small, medium and large.

This won't get the exact sizes but applies dynamic scaling and will actually modify the picture making it more bold when smaller and less bold when larger.

This method was used by Apple in [WWDC 2020](https://developer.apple.com/videos/play/wwdc2020/10207/)

