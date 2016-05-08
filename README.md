<p align="center">
![Render Logo](Doc/logo.png)


[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![Build](https://img.shields.io/badge/build-passing-green.svg?style=flat)](#)
[![Platform](https://img.shields.io/badge/platform-ios-lightgrey.svg?style=flat)](#)
[![Build](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://opensource.org/licenses/MIT)

*React-inspired swift library for writing UIKit UIs which are functions of their state.*

#Why

**Render** lets us write our UIs as a pure function of their state.


Right now we write UIs by poking at them, manually mutating their properties when something changes, adding and removing views, etc. This is fragile and error-prone. Some tools exist to lessen the pain, but they can only go so far. UIs are big, messy, mutable, stateful bags of sadness.

**Render** let us describe our entire UI for a given state, and then it does the hard work of figuring out what needs to change. It abstracts all the fragile, error-prone code out away from us. 

## Installation

### Carthage



To install Carthage, run (using Homebrew):

```bash
$ brew update
$ brew install carthage
```

Then add the following line to your `Cartfile`:

```
github "alexdrone/Render" "master"    
```

#TL;DR

Render's building blocks are *Components* (described in the protocol `ComponentViewType`).
Despite virtually any `UIView` object can be a component (as long as it conforms to the above-cited protocol),
Render's core functionalities are exposed by the two main Component base classes: `ComponentView` and `StaticComponentView` (optimised for components that have a static view hierarchy).

Render layout engine is based on [FlexboxLayout](https://github.com/alexdrone/FlexboxLayout).

This is what a component (and its state) would look like:


```swift

struct Album: ComponentStateType {
	let title: String
	let artist: String
	let cover: UIImage  
	let featured: Bool
}

// COMPONENT
class AlbumComponentView: ComponentView {
    
    // the component state.
    var album: Album? {
        return self.state as? Album
    }
    
    // View as function of the state.
    override func construct() -> ComponentNodeType {
            
        return ComponentNode<UIView>().configure({
        		$0.style.flexDirection = self.album.featured ? .Row : .Column
            	$0.backgroundColor = UIColor.blackColor()

        }).children([
            
            ComponentNode<UIImageView>().configure({
				$0.image = self.album?.cover
				let size = self.album.featured ? self.parentSize.width : 48.0
				$0.style.dimensions = (size, size)
            }),
            
            ComponentNode<UIView>().configure({ 
            		$0.style.flexDirection = .Column
            		$0.style.margin = (8.0, 8.0, 8.0, 8.0, 0.0, 0.0)
                
            }).children([
                
                ComponentNode<UILabel>().configure({ 
                		$0.text = self.album?.title ?? "None"
                		$0.font = UIFont.systemFontOfSize(18.0, weight: UIFontWeightBold)
                		$0.textColor = UIColor.whiteColor()
                }),
                
                ComponentNode<UILabel>().configure({ view in
                		$0.text = self.album?.artist ?? "Uknown Artist"
                		$0.font = UIFont.systemFontOfSize(12.0, weight: UIFontWeightLight)
                		$0.textColor = UIColor.whiteColor()                		
                })
            ]),
         
            // This node will be part of the tree only when featured == false. *
            when(!self.album?.featured, ComponentNode<UILabel>().configure({ view in
                $0.style.justifyContent = .FlexEnd
                $0.text = "2016"
                $0.textColor = UIColor.whiteColor()
            }))

        ])
    }
    
}

```
<sup> * This can also be accomplished with a `StaticComponentView` by setting the `hidden` property in the configuration closure. </sup>


The view description is defined by the `construct()` method.
`ComponentNode<T>` is an abstaction around views of any sort that knows how to build, configure and layout the view when necessary.
Every time `renderComponent()` is called, a new tree is constructed, compared to the existing tree and only the required changes to the actual view hierarchy are performed - *if you have a static view hierarchy, you might want to inherit from `StaticComponentView` to skip this part of the rendering* . Also the `configure` closure passed as argument is re-applied to every view defined in the `construct()` method and the layout is re-computed based on the nodes' flexbox attributes. 

The component above would render to:

<p align="center">
<img src="Doc/render.jpg" width="900">

```swift

let albumComponent = AlbumComponentView()
albumComponentView.renderComponent()
```

