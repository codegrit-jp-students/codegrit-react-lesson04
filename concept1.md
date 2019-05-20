# コンポーネントライフサイクルイベントの種類

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