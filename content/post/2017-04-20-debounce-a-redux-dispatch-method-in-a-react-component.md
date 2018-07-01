---
id: 4266
title: Debounce a redux dispatch method in a react component
date: 2017-04-20T12:46:18+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4266
permalink: /2017/04/20/debounce-a-redux-dispatch-method-in-a-react-component/
dsq_thread_id:
  - "5743813164"
image: /wp-content/uploads/2017/03/Redux-and-React.png
categories:
  - React
  - Redux
tags:
  - delay
  - dispatch
  - execution
  - filter
  - limit
  - react
  - redux
  - request
  - search
  - throttle
---
Tightly connected reactivity in a react application has the side effect that it is sometimes necessary to delay the execution of a method. Assume you have a search input field that filters elements while typing, every field input creates a search request. In order to get rid of unnecessary search requests you have to wait until a user has finished typing and then start the search. To make this work without a search button, you have to intercept repeating executions (debounce) of the search method within a specified time frame and delay the execution of the last call.
<!--more-->
Here's a simple example of a react search component where the redux dispatch method will be debounced.

[code]
import React from 'react'
import { Card, CardText, TextField } from 'material-ui'
import { setUserFilter } from '../actions'
import { debounce } from 'lodash'

class UserSearch extends React.Component {

  constructor(props){
    super(props)
    this.updateFilter = debounce(this.updateFilter, 500)
  }

  updateFilter(){
    let { dispatch } = this.props
    let { filter } = this.refs
    dispatch(setUserFilter(filter.getValue()))
  }

  render() {
    return <Card>
      <CardText>

        <TextField
        ref="filter"
        onChange={this.updateFilter.bind(this)} />

        ...

      </CardText>
    </Card>
  }
}
[/code]

The search is only dispatched if the last call of the update filter method is older than 500ms.

Source: [Stack Overflow - Perform debounce in React.js](http://stackoverflow.com/questions/23123138/perform-debounce-in-react-js)
