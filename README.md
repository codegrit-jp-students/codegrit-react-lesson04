# レッスン4. コンポーネントライフサイクル

## サンプルコード

このレッスンのサンプルコードは以下のリポジトリにあります。

[codegrit-react-lesson04-samples](https://github.com/codegrit-jp-students/codegrit-react-lesson04-samples)

## コンポーネントライフサイクルとは

レッスン3までで、コンポーネントの状態(state)やプロパティ(props)について学んできました。ここまでで学んできたことでも多くのことを実現できます。しかしコンポーネントによっては例えばブラウザのサイズに応じて表示を切り替えたり、スクロールされた時に新しいデータを読み込みたい、などの操作を行いたい場合があります。こうした時のためにReactではコンポーネントライフサイクルイベントというものが用意されています。また、これまで利用してきたrender()もライフサイクルイベントの一つです。

コンポーネントには大きく分けて4つのライフサイクルがあります。

- マウント中(Mounting) - コンポーネントが読み込まれ、表示されている状態
- アップデート中(Updating) - コンポーネントが新しいPropsを受け取ったり、Stateが変更され、表示のアップデートが必要、あるいはアップデートが完了した状態
- アンマウント中(Unmounting) - コンポーネントが非表示となるまで、非表示となった状態
- エラー処理(Error Handling) - renderイベント呼び出し中にエラーが発生した状態

この3つのうち、MountとUnmountの状態は1回しか起こりません。しかし、Updateについてはコンポーネントがアップデートされる度に呼び出されるので複数回この状態となることがあります。

これらの4つのライフサイクルにおいて、様々なイベントが用意されています。この流れは非常に重要ですので、今後も何度も確認して覚えるようにしましょう。

## マウント中のイベント

Mount時には以下の順番でイベントが発生します。

1. constructor()
2. static getDerivedStateFromProps()
3. render()
4. componentDidMount()

## アップデート中のイベント

アップデート中は以下の順番でイベントが発生します。

1. static getDerivedStateFromProps()
2. shouldComponentUpdate()
3. render()
4. getSnapshotBeforeUpdate()
5. componentDidUpdate()

## アンマウント中のイベント

アンマウント中は以下の順番でイベントが発生します。

1. componentWillUnmount()

## エラー処理に関するイベント

エラー処理に関するイベントはReact16で新しく加わりました。

1. static getDerivedStateFromError()
2. componentDidCatch()

## 廃止予定のイベントと新しく加わったイベント

React16.3では、これまで利用されていたいくつかのライフサイクルイベントが廃止予定となり、代わって新しく加わったイベントがあります。しかし、今後しばらくの間は古いReactのコードで廃止されたイベントを見る機会もあるかと思いますので簡単に触れます。

### 廃止予定のイベント

1. componentWillMount()

このイベントは、本来コンポーネントがマウントされる直前に呼び出されていましたが、renderイベントがループされてしまう問題が発生しやすかったため、React17において廃止され、UNSAFE_componentWillMountという名前に変更されることが決まっています。

このイベントで出来ることは、constructorとcomponentDidMountイベントを利用して代替出来ます。

2. componentWillReceiveProps()

このイベントは、コンポーネントの最初のマウント時には呼び出されません。マウント後、コンポーネントが新しいPropsを受け取る直前に呼び出されます。このイベントはよくバグやアップデートが上手くいかない問題が発生していたためReact17での廃止が決まっています。

3. componentWillUpdate()

このイベントは、コンポーネントの最初のマウント時には呼び出されません。マウント後、コンポーネントが新しいPropsを受け取ったり、Stateが変更された後、再度renderイベントが呼び出される直前に呼び出されます。componentWillUpdate内でthis.setStateを呼ぶと、コンポーネントがアップデートされる前に再度componentWillUpdateが呼び出されてしまいループが発生する危険があります。

このイベントはcomponentDidUpdate()とgetSnapshotBeforeUpdate()を利用することで代替することができます。

### 追加されたイベント

1. static getDerivedStateFromProps()
2. getSnapshotBeforeUpdate()

これらの2つについては、後述の各イベントの解説の部分をご覧ください。

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

## Error処理用のイベント

以下の2つのイベントは

### static getDerivedStateFromError()

```js
static getDerivedStateFromError(error)
```

このイベントはrender時に何らかのイベントがあった場合に、stateを変更して、エラー関連の表示を行いたいという場合に利用します。この際、ErrorBoundary.jsのようにErrorをキャッチするためのコンポーネントを作成し、エラー発生可能性のある、childrenにあたるコンポーネントを囲います。

```js
// src/ErrorBoundary.js

import React, { Component } from 'react';

export default class extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // この中で外部サーバーにエラー情報を送信するようなことをします。
    console.log(error);
  }

  render() {
    if (this.state.hasError) {
      return <h1>エラーが発生しました！</h1>;
    }

    return this.props.children;
  }
}
```

例えば上記のように、ErrorBoundary.jsを設定します。その上で、このErrorBoundaryを利用します。

```js
// src/GetDerivedErrorSample.js

import React, { Component } from 'react';
import ErrorBoundary from './ErrorBoundary';
import { fetchMentorsWithfailure } from './ApiRequestMock';

const MentorsComponent = ({ mentors }) => {
  if (mentors === null) {
    throw new Error("メンターが取得できていません！"); // render内部でエラーが起こった場合を想定。
  }
  return (
    <p>メンターが{mentors.length}人います。</p>
  );
}

export default class extends Component {
  state = {
    isLoading: true,
    mentors: null,
  }
  async componentDidMount() {
    try {
      const mentors = await fetchMentorsWithfailure();
      this.setState({
        isLoading: false,
        mentors
      });
    } catch (err) {
      this.setState({
        isLoading: false,
      })
    }
  }

  render() {
    const { isLoading, mentors } = this.state;
    if (isLoading) return <p>メンター情報を取得しています。</p>
    // ErrorBoundaryコンポーネントのchildrenにエラーをキャッチしたいコンポーネントを渡します。
    return (
      <ErrorBoundary>
        <p>エラーの場合その旨が表示されます。</p>
        <MentorsComponent mentors={mentors}/>
      </ErrorBoundary>
    );
  }
}
```

### 5. componentDidCatch()

```js
componentDidCatch(error, info)
```

このイベントは、主に発生したエラーのログを残したい時に利用します。例えば、Sentry、Bugsnagのようなエラートラッキングアプリを利用している場合、発生したエラーの情報をこうしたアプリに送信するようなことができます。

プログラマーは、このようにエラー情報を集め、そのエラーを解決していくことでアプリのバグ修正を行います。

引数に入るerrorとinfoにはそれぞれ以下のような情報が含まれます。

- error - エラーの中身
- info - どのコンポーネントがエラーを出したのか。(info.componentStackで呼び出せる。)

```js
import React, { Component } from 'react'
import { logErrorToBugsnag } from 'errorHandlingUtils'

export default class extends Component {
  
  state = {
    hasError: false
  }

  static getDerivedStateFromError(error) {
    return {
      hasError: true
    }
  }

  componentDidCatch(error, info) {
    // この部分でBugsnagにエラー情報を送信しています。
    logErrorToBugsnag(error, info.componentStack)
  }

  render() {
    const { hasError } = this.state;
    let errorMessage = <div />
    if (hasError) {
      errorMessage = <p>エラーが発生しました。</p>
    }
    return (
      <main>
        {errorMessage}
        省略...
      </main>
    );
  }
}
```

## 更に学ぼう

- [React公式 - stateとライフサイクル](https://ja.reactjs.org/docs/state-and-lifecycle.html)
- [React公式 - Error Boundary](https://ja.reactjs.org/docs/error-boundaries.html)