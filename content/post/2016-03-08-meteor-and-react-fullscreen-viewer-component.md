---
id: 3824
title: 'Meteor and React: Fullscreen Viewer Component'
date: 2016-03-08T15:51:29+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3824
permalink: /2016/03/08/meteor-and-react-fullscreen-viewer-component/
dsq_thread_id:
  - "4644749767"
image: /wp-content/uploads/2016/03/meteor-react-logo-1200x201.png
categories:
  - React
tags:
  - button
  - children
  - component
  - display
  - easy
  - editor
  - fullscreen
  - fullview
  - icon
  - meteor
  - react
  - view
  - viewer
---
Another post as result of my Meteor React experience.
This is a simple approach to build a fullscreen viewer for any React component.
<!--more-->
I assume you've added the `bootstrap` and `react` package to your project.

The fullscreen viewer component is very simple. It wraps the children in a div tag and adds an Icon to the upper right of this container.

[code lang="js"]
FullscreenContainer = React.createClass({
  getInitialState(){
    return{
      screenClass: 'display-normal'
    };
  },
  switchFullscreen(){
    if(this.state.screenClass != 'display-fullscreen'){
      this.setState({screenClass: 'display-fullscreen'});
    }else{
      this.setState({screenClass: 'display-normal'});
    }
  },
  getIcon(){
    if(this.state.screenClass === 'display-fullscreen'){
      return <i onClick={this.switchFullscreen} className="glyphicon screen-icon glyphicon-remove" />
    }else{
      return <i onClick={this.switchFullscreen} className="glyphicon screen-icon glyphicon-fullscreen" />
    }
  },
  render() {
    return <div className={this.state.screenClass}>
    {this.getIcon()}
    {this.props.children}
    </div>
  }
});
[/code]

Next add this css code. The fullscreen display is works with css only!

[code lang="css"]
.display-fullscreen {
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  z-index:1000;
  background-color: white;
  overflow: hidden;
}

.screen-icon{
  float: right;
  background: white;
  right: 0;
  font-size: 20px;
  z-index:1001;
}
[/code]

This css classes will be applied to the fullscreen container and make it overlay the window.

Here's and example how you can use this component in another view:

[code lang="js"]

...

render() {
     return <FullscreenContainer>
        
        ...

        </FullscreenContainer>;
    }

...

});
[/code]

In my case the final result for a markdown editor looked like this:

<img src="https://janikvonrotz.ch/wp-content/uploads/2016/03/fullscreen-container-default-1024x672.png" alt="fullscreen container default" width="840" height="551" class="aligncenter size-large wp-image-3860" />

Before clicking on the fullscreen button.

<img src="https://janikvonrotz.ch/wp-content/uploads/2016/03/fullscreen-container-display-1024x426.png" alt="fullscreen container display" width="840" height="349" class="aligncenter size-large wp-image-3861" />

After clicking.