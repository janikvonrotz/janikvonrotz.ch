---
id: 3830
title: 'Meteor and React: Bootstrap Modal'
date: 2016-03-05T12:42:05+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3830
permalink: /2016/03/05/meteor-and-react-bootstrap-modal/
dsq_thread_id:
  - "4637306001"
image: /wp-content/uploads/2016/03/meteor-react-logo-1200x201.png
categories:
  - React
tags:
  - bootstrap
  - component
  - confirm
  - css
  - dialog
  - display
  - hide
  - jsx
  - meteor
  - modal
  - react
  - show
  - toggle
  - twitter
---
I would like to share my great experience I had with Meteor and React. This time I want to show how I've built a bootstrap modal component. There are already a lot of solutions out there, but none of them were suitable.

Here's how you can get a neat bootstrap modal for your Meteor React project.
<!--more-->
I assume you already have added the `bootstrap`, `classnames` and `react` package to your project.

[code lang="js"]
Modal = React.createClass({
  renderFooter(){
    if(!this.props.footer){
      return &lt;div className=&quot;modal-footer&quot;&gt;
      &lt;Button style=&quot;primary&quot; onClick={this.props.onConfirm}&gt;{this.props.confirmLabel}&lt;/Button&gt;
      &lt;Button style=&quot;default&quot; onClick={this.props.onCancel}&gt;{this.props.cancelLabel}&lt;/Button&gt;
      &lt;/div&gt;
    }else{
      return &lt;div className=&quot;modal-footer&quot;&gt;{this.props.footer}&lt;/div&gt;
    }
  },
  render() {
    var modalClassName = classNames({
      &quot;modal&quot;: true,
      &quot;show-modal&quot;: this.props.showModal
    });
    return &lt;div className={modalClassName}&gt;
      &lt;div className=&quot;modal-dialog&quot;&gt;
        &lt;div className=&quot;modal-content&quot;&gt;
          &lt;div className=&quot;modal-header&quot;&gt;
            &lt;Button className=&quot;close&quot; onClick={this.props.onCancel} ariaLabel=&quot;Close&quot;&gt;&lt;span aria-hidden=&quot;true&quot;&gt;&amp;times;&lt;/span&gt;&lt;/Button&gt;
            &lt;h4 className=&quot;modal-title&quot;&gt;{this.props.title}&lt;/h4&gt;
          &lt;/div&gt;
          &lt;div className=&quot;modal-body&quot;&gt;
            {this.props.children}
          &lt;/div&gt;
          {this.renderFooter()}
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
  }
});
[/code]

The Button tag of course is another React component that you have to build yourself.


To toggle the visibility of the modal dialog simply add this css class.

[code lang="css"]
.show-modal{
    display: block;
}
[/code]

Here's and excerpt of the necessary code to run the component:

[code lang="js"]
YourComponent = React.createClass({

...

getInitialState(){
   return{
     showModal: false
   };
},
toggleModal(){
  this.setState({showModal: !this.state.showModal});
},
render(){
  &lt;Modal
    showModal={this.state.showModal}
    title=&quot;Confirm&quot;
    onCancel={this.toggleModal}
    cancelLabel=&quot;Cancel&quot;
    onConfirm={this.deleteItem}
    confirmLabel=&quot;Delete&quot;
  &gt;&lt;p&gt;Please confirm the deletion of item: {this.data.item.label}&lt;/p&gt;&lt;/Modal&gt;
}
...

});
[/code]

The final result:

<img src="https://janikvonrotz.ch/wp-content/uploads/2016/03/confirm-modal-react-300x97.png" alt="confirm modal react" width="300" height="97" class="aligncenter size-medium wp-image-3857" />