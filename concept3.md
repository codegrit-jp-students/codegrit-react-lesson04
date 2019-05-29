# Error Boundary

## エラー対応のためのイベント

React16では、レンダー時に発生するエラーへの対応を行うためのイベントが紹介されました。一般的にErrorBoundaryというコンポーネントを作成し、その中にエラーの発生しそうなコンポーネントを格納して利用します。

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

### componentDidCatch()

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
