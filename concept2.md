# 各イベントの解説

## よく使われるイベントと解説

以下ではコンポーネントライフサイクルにおける各イベントの中でも特によく使われるものについて詳しくみていきます。

### componentDidMount()

```js
componentDidMount()
```

componentDidMountは名前の通り、コンポーネントがマウントされた直後に呼び出されます。

よくある利用シーン: 

- APIからデータを読み込む
- addEventListnerなどを設定する

例えば以下の例では、マウント後にfetch APIを利用してデータを読み込んでいます。

```js
// src/ComponentDidMountSample.js

import React, { Component } from 'react';
import { fetchMentorsWithfailure } from './ApiRequestMock';

export default class extends Component {
  state = {
    isLoading: true, 
    mentors: null,
    errorMessage: null
  }
  async componentDidMount() {
    try {
      const mentors = await fetchMentorsWithfailure();
      this.setState({
        isLoading: false,
        mentors
      });
    } catch(err) {
      this.setState({
        isLoading: false,
        errorMessage: err.message,
      })
    }
  }
  render() {
    const { isLoading, mentors, errorMessage } = this.state;
    if (isLoading) return <p>メンター情報を取得しています。</p>
    if (mentors === null && errorMessage) return <p>エラーが発生しました。</p> 
    if (mentors.length > 0) {
      return <p>メンターが{mentors.length}人います。</p>
    }
    return <p>メンターは0人です。</p>
  }
}

```

### componentDidUpdate()

```js
componentDidUpdate(prevProps, prevState, snapshot)
```

componentDidUpdateは、コンポーネントがアップデートされ、結果がレンダリングされた後に呼び出されます。後述のgetSnapshotBeforeUpdateを利用している場合、3つ目の引数であるsnapshotを利用してその結果を利用することができます。ただしsnapshotを利用することは稀です。

よくある利用シーン:

- チャットなどで他のスレッドが選択されデータをAPIから読み込みたい場合。

```js
import React, { Component } from 'react'

export default class extends Component {
  componentDidUpdate(prevProps) {
    if (prevProps.activeConversationId !== this.props.activeConversationId) {
      this.fetchData(this.props.activeConversationId)
    }
  }
  fetchData = (conversationId) => {
    ...
  }
  render() {
    省略...
  }
}
```

### componentWillUnmount()

componentWillUnmountはコンポーネントがアンマウントされる直前に呼び出されます。componentWillUnmountの直後にはコンポーネントがアンマウントされるので、Stateもリセットされます。この中ではsetStateの呼び出しはしないようにしましょう。

よくある利用シーン:

- componentDidMountでされたaddEventListnerなどの登録を解除する

```js
// src/ComponentWillUnmountSample.js

import React, { Component } from 'react';

export default class extends Component {
  state = {
    resized: false
  }

  componentDidMount() {
    window.addEventListener("resize", this.handleResize);
  }

  componentWillMount() {
    window.removeEventListener("resize", this.handleResize);
  }

  handleResize = () => {
    this.setState({
      resized: true
    })
  }

  render() {
    const { resized } = this.state;
    if (resized) return <p>リサイズされました。</p>
    return <p>リサイズされていません。</p>

  }
}
```

## 使用機会の少ないイベントと解説

### static getDerivedStateFromProps()

```js
static getDerivedStateFromProps(props, state)
```

このイベントはrenderイベントが発生する直前に起こります。マウント時とアップデート時両方に呼び出されます。このイベントの目的はただ一つで、Propsに依存するstateが存在する場合です。その場合にPropsが変更されてからStateを変更したいという時にこのイベントを利用します。

このイベントはバグを引き起こしやすいため、他の方法でどうしてもやりたいことが実現できない時にドキュメントを読みながら利用することをおすすめします。こういうイベントがあるとだけ覚えておきましょう。

どういったバグが起きるかについてはReactのドキュメントに詳しく書かれています。(You Probably Don't Need Derived State)[https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization]

よくある利用シーン:

- フォームのインプットを選択されているユーザーが変わった時にアップデートしたい場合

以下の例では、例えば管理画面で複数のユーザーを編集している時を想定しています。編集しているユーザーが変更された際に、stateをそのユーザーのユーザーIDと現在のメールアドレスに変更します。またメールアドレスを書き換えようとしている場合にはhandleChangeを利用して、書きかけのメールアドレスをStateに入れ直しています。

getDerivedStateの中ではsetStateではなく、シンプルにreturn { プロパティ: 値 }でstateを変更できることにも注目しましょう。

メールアドレスの編集

```js
import React, { Component } from 'react'

class EmailInput extends Component {
  state = {
    userId: this.props.userId,
    email: this.props.initialEmail
  }
  static getDerivedStateFromProps(props, state) {
    // if文でこれまで編集していたユーザーから、他のユーザーに切り替えたかどうかのチェックを行う。
    if (this.state.userId !== props.userId) {
      return {
        userID: props.userID, // 変更されたユーザーのユーザーID
        email: props.initialEmail // 変更されたユーザーの現在のメールアドレス
      };
    }
    return null // Stateに変更が加わらない場合はnullを返すこと。
  }
  handleChange(e) {
    e.preventDefault()
    this.setState({
      email: e.target.value;
    })
  }
  render() {
    省略...
  }
}
```

### shouldComponentUpdate()

```js
shouldComponentUpdate(nextProps, nextState)
```

shouldComponentUpdate()は、アップデートの際、renderイベントの直前に呼び出されます。shouldComponentUpdateはBoolean値(trueかfalse)を返し、デフォルトではtrueとなっています。

shouldComponentUpdateがfalseになっている場合には、その後のrenderイベントとcomponentDidUpdateイベントは呼び出されません。このイベントは、Reactアプリのパフォーマンスを向上させたい時のみ使うもので初心者のうちは使う必要がありません。また初心者のうちに使うとバグの原因となりやすいのでReactに十分に慣れ、パフォーマンスの重要となるアプリを開発する機会に挑戦することをおすすめします。現時点では、こうしたイベントがあるという理解のみで十分です。

### getSnapshotBeforeUpdate()

```js
getSnapshotBeforeUpdate(prevProps, prevState)
```

getSnapshotBeforeUpdateは、アップデート中のrenderイベントの直後に発生します。このイベントは直前にrenderされたDOMの情報を利用して、新しくrenderされたDOMのアップデートをしたい場合に利用します。

よくある利用シーン
- チャットで、新しいメッセージを読み込んだ後に、メッセージが読み込まれる前のメッセージ部分までスクロール位置を戻したい場合。

このイベントも上級者向けのため、使う機会があるまではあることを知っていれば大丈夫です。使う機会があればドキュメントを読んで理解した上で使いましょう。