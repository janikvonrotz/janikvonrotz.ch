---
id: 4204
title: Make your Redux React app multilingual
date: 2017-03-02T12:51:10+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4204
permalink: /2017/03/02/make-your-redux-react-app-multilingual/
dsq_thread_id:
  - "5596829010"
image: /wp-content/uploads/2017/03/Redux-and-React.png
categories:
  - React
  - Redux
tags:
  - client
  - component
  - i18n
  - internationalization
  - multilanguage
  - react
  - redux
  - state
  - store
  - translate
---
For my current React app in development I'm using Redux to manage the client state. As this is the first time using Redux I'm not quite sure what to put in the store yet. But I've decided to give the multilingual labels and wordings a try. Below you'll find a simple approach on how I made my app multilingual using the Redux client state to store the translated content. I assume you are familiar with Redux and React.
<!--more-->
The `i18n` state reducer contains all translation and puts the default language content into the store.

**reducers/i18n.js**

[code]
let phrases = {
  de: {
    button: {
      remove: 'Löschen',
      update: 'Speichern',
      commit: 'Neue Version',
      add_router: 'Router hinzufügen',
      search: 'Suche',
      cancel: 'Abrechen',
    }
  },
  en: {
    button: {
      remove: 'Delete',
      update: 'Save',
      commit: 'Commit',
      add_router: 'Add Router'
    }
  }
}

export default (state = phrases.de, action) => {
  switch (action.type) {
    case 'SWITCH_LANGUAGE':
      return phrases[action.language]
    default:
      return state
  }
}
[/code]

To get f.g. the translated button labels simply map the `i18n` state as prop on the component.

**components/RouterSearch.js**

[code]
...
import React from 'react'
import { connect } from 'react-redux'

class RouterSearch extends React.Component {

  ...

  render() {
    let { i18n } = this.props

    return <Card>
      <CardText>

        <TextField
        floatingLabelText={ i18n.button.search }

        ...

      </CardText>
    </Card>
  }
}

const mapStateToProps = (state) => {
  return {
    i18n: state.i18n,
  }
}
export default connect(mapStateToProps)(RouterSearch)
[/code]

Switching the language is easily done by setting the `language` state.

**actions.js**

[code]
...
export const switchLanguage = (language) => {
  return {
    type: 'SWITCH_LANGUAGE',
    language
  }
}
...
[/code]

Obviously the are a lot cons to this method. First, there is no fallback for not translated content as you access to `i18n` state directly. Second, putting multilingual content into the store is probably not a good idea in the first place. What do you think? Are there better options?