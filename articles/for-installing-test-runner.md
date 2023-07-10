---
title: "[Storybook]Test runner導入でハマった・工夫したところ4つをまとめてみた。"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Storybook", "Tech", "Jest"]
published: false
---

こんにちは、アルダグラムのエンジニアの影山です。

ダイエットのために食事改善を始めました。ただ、出社時に甘いものが欲しくなりタイ焼きを食べたら両脇のエンジニアから「ダイエットとは」を5分ほど問われました。今後のおやつは隠れて食べようと思います。

今回は KANNA（弊社アプリ）の Webフロントエンドに Test runner を導入することになりました。

すんなりいくかなと思ったのですが、いくつか迷い道を渡りながら実装したので、その知見をまとめていきます。

Test runner 導入に至った背景に関しては、こちらの記事から確認できます。

記事：[Webフロントエンドに Integration Test を導入するに至った諸々の話](https://zenn.dev/aldagram_tech/articles/kanna-integration-test)

## Storybook のバージョン

弊社では Storybook を既に導入しており、addon を含めて下記のようなバージョンになっています。

```json
"@storybook/addon-actions": "^6.5.10",
"@storybook/addon-essentials": "^6.5.10",
"@storybook/addon-interactions": "^6.5.10",
"@storybook/addon-links": "^6.5.10",
"@storybook/builder-webpack5": "^6.5.10",
"@storybook/manager-webpack5": "^6.5.10",
"@storybook/react": "^6.5.10",
"@storybook/testing-library": "^0.0.13",
"@storybook/jest": "^0.1.0",
"@storybook/test-runner": "^0.9.4",
```

2023/07現在における最新バージョンであるv7 では test-runner のバージョンが `^0.10.0` になると思うので、注意です。 `@storybook/jest` は、実際のstoriesファイルの `expect` で使うのでほぼ必須になると思います。

| Test runner version | Storybook version |
| --- | --- |
| ^0.10.0 | ^7.0.0 |
| ~0.9.4 | ^6.4.0 |

参考：https://github.com/storybookjs/test-runner#storybook-compatibility

## 章立て

1. [ハマり]パッケージ依存関係で別のspecファイルの型エラーが起きる
2. [工夫]pagesのstoryのみをテストしたい
3. [工夫]言語別にテストがしたいが、どうすればいい？
4. [ハマり]CIで `Failed to build the preview` エラーが出た

### [ハマり]パッケージ依存関係で別のspecファイルの型エラーが起きる

課題：（これはプロダクト依存な部分もあるのですが）、別のspecファイルで、 `NextApiRequest` ,  `NextApiResponse` を返す部分があるのですが、そこで使われている `http-server` との return 値がエラーを吐いていました。

```json
error TS2344: Type 'NextApiRequest' does not satisfy the constraint 'Request<ParamsDictionary, any, any, ParsedQs, Record<string, any>>'.
```

直接的な原因は掴めていないのですが、事象としては下記と同様でした。

[Parameters mismatch Typescript definitions · Issue #245 · eugef/node-mocks-http](https://github.com/eugef/node-mocks-http/issues/245#issuecomment-926390977)

解決策：specファイルの型の部分を

```tsx
import httpMocks, { createRequest, createResponse } from 'node-mocks-http'

type ApiRequest = NextApiRequest & ReturnType<typeof createRequest>
type APiResponse = NextApiResponse & ReturnType<typeof createResponse>

describe('hoge', () => {
  describe('fuga', () => {
    const mockReq = httpMocks.createRequest<ApiRequest>({
      // ...
    })
  }
}
```

と、 `node-mocks-http` の型を併せて使うことでエラーが解消されました。

### [工夫]pagesのstoryのみをテストしたい

課題：KANNA では既にページごとの stories ファイルが存在するため、ページごとに interactions test を追加する方針にしました。しかし、Test runner の初期セットアップだと、KANNA で採用している　AtomicDesign の Atoms, Molecules, Organisms などの pages 以外の stories ファイルもテストしまうため、こちらは最初は省きたかったのです。

解決策： [test-runner-jest.config.js](https://storybook.js.org/addons/@storybook/test-runner#:~:text=shard%3D1/3-,Ejecting%20configuration,-The%20test%20runner) を利用することで、 `testMatch` オプションで pages.stories のみをテストするようにしました。この config は、 `test-runner` コマンド実行時に適用されるもので、 jest-playwright と jest の２つのオプションを受け付けます。

```jsx
const { getJestConfig } = require('@storybook/test-runner')

module.exports = {
  // The default configuration comes from @storybook/test-runner
  ...getJestConfig(),
  /** Add your own overrides below
   * @see https://jestjs.io/docs/configuration
   */
  testMatch: [
    "**/pages.stories/**/*.stories.@(js|jsx|ts|tsx)",
  ],
}
```

また、一時的にテストしたくない stories に対しては  `testPathIgnorePatterns` を使って回避できます。

```jsx
...
  testMatch: [
    "**/pages.stories/**/*.stories.@(js|jsx|ts|tsx)",
  ],
  testPathIgnorePatterns: [
    // 修正中のため、一時的に該当ファイルをignoreしています

    // Unable to find an element by: [data-testid="btn-delete"]
    ".+\\/pages\\.stories\\/hoge\\/\\/index\\.stories\\.tsx",
  ],
}
```

これにより「Test runner 導入しようと思ったけど、エラーが多すぎて修正コストに時間がめっちゃかかるやん..」みたいなリスクを一時的に避けられますね。

### [工夫]言語別にテストがしたいが、どうすればいい？

注意：これはベストプラクティスが見つけられておらず、現時点での話です。

KANNAは現在、日本語・英語・タイ語・スペイン語をサポートしており、UI 上でも海外ユーザー用にいくつか項目を省く or 追加したりしています。例えばカナ入力は海外では省かれています。

課題：日本語の場合、カナ入力を含めてユーザー登録の interactions test をしたい。日本語以外の場合、カナ入力を省いたユーザー登録をしたい。

最初の解決策： `hoge/interactions/ja` , `hoge/interactions/en` とディレクトリを切ってテストを書いていく。

愚直に書いていくスタイルです。KANNAでは i18n を利用しており、それを各ストーリーの実行ごとに切り替えていけば良い？と思っておりました。

やることとしては、

1. decorators に i18n を修正して**言語の**切り替えをする
2.  `language`,  `country` というカナ入力フォームを出す・出さない分岐を扱うデータをユーザー側(以下viewerとします)で保持しているため、これを各ファイルで設定して**項目の**切り替えを行う。

decorators は具体的にはこんな感じです。 decorators のなかで viewer も切り替えています。

下記のような処理が en, ja 両方で作られるイメージです。

```jsx
hogehoge.decorators = [
  (
    Story: ComponentStory<typeof Page>,
    context: StoryContext<ReactFramework, Args>
  ) => {
    useEffect(() => {
      i18n.changeLanguage('en') // jaの場合 'ja' 指定
      return () => {
        i18n.changeLanguage(context.globals.locale)
      }
    }, [context.globals.locale])

    const [viewer, setViewer] = useState<Viewer>(
      new Viewer({
        // ...ユーザーのデータ色々,
        language: SupportLanguages.En, // jaの場合　Jp　指定
        country: CompanyCountries.Us // jaの場合　Ja　指定
      })
    )
    return (
      <I18nextProvider i18n={i18n}>
        <ViewerContext.Provider value={{viewer, setViewer}}>
          <Story />
        </ViewerContext.Provider>
      </I18nextProvider>
    )
  }
],
hogehoge.play = async () => {
  for (const testData of formTestData) {
    // いろんなテストコード
  }
  await expect(hoge).toBeEnabled()
}
```

実装上は問題なく、日本語とそれ以外で項目が一緒ならこのような分岐も必要ないです。

ただ、そうでない場合 interactions test のために、毎回ディレクトリを切って i18n の切り替えなどが必要になるので手間。というデメリットがあり、もっと簡単にできないかなーっと別のエンジニアと議論したりしていました。

最終的な解決策：preview.tsx でグローバルに扱っている `locale context` を、各stories ファイルの play関数内で見て実行時の処理を変える。

Storybook では、globals というストーリがレンダリングされるビューポートと背景を制御するツールバーアドオンが実装されています。

[Toolbars & globals](https://storybook.js.org/docs/react/essentials/toolbars-and-globals)

globals はカスタマイズができるため、globals で設定されている言語を見て、play関数内の処理を変えられるというわけです。

１：preview.tsx で globalTypes を設定します。

```jsx
export const globalTypes = {
  storybookLocale: {
    name: 'storybookLocale',
    description: 'Internationalization locale',
    defaultValue: getStorybookLocaleHogeHoge(),
    toolbar: {
      icon: 'globe',
      items: [
        { value: 'en', right: '🇺🇸', title: 'English' },
        { value: 'ja', right: '🇯🇵', title: 'Japan' },
        // hoge
      ]
    }
  }
}
```

設定した globals は、 preview.tsx の decorators に context として渡します。

```tsx
export type StorybookContext = {
  globals: {
    storybookLocale: string
  }
}

export const decorators = [
  hogehoge,
  (Story: ComponentStory<any>, context: StorybookContext) => {
    // hoge...
    const userLanguage = context.globals.storybookLocale.toUpperCase()
    const companyCountry = (language => {
      switch (language) {
        case SupportLanguages.Ja:
          return CompanyCountries.Jp
        // 他のケース...
        default:
          return CompanyCountries.Us
      }
    })(userLanguage)
    const [viewer, setViewer] = useState<Viewer>(
      new Viewer({
        // ...ユーザーのデータ色々,
        userLanguage: userLanguage as SupportLanguages,
        companyCountry
      })
    )
  //...
  }
]
```

これで各 stories を実行する時、  `context.globals.storybookLocale` を play関数内で見ることで、処理を変えられます。

```tsx
hogehoge.play = async context => {
  const play = async (
    formTestData: {
      dataTestId: string
      inputValue: string
      expectedValue: string
    }[]
  ) => {
    for (const testData of formTestData) {
      // いろんなテストコード
    }
    await expect(hoge).toBeEnabled()
  }

  if (context.globals.storybookLocale === 'ja') {
    await play(formTestDataJa)
  } else {
    await play(formTestData)
  }
}
```

実際には、既に `context.globals.storybookLocale` が KANNA 上で実装されていたので、それを利用するだけで問題なかったです。早く気づければよかった。反省。

### [ハマり]CIで `Failed to build the preview` エラーが出た。

言語別の実装も完了したので、CI で回そうと思っていました。

ローカルでは通っているし大丈夫やろと思っていたら、こんなエラーが出ました。

```tsx
ERR! => Failed to build the preview
ERR! export 'createChannel' (imported as 'createChannel') was not found in '@storybook/channel-postmessage' (possible exports: KEY, PostmsgTransport, default)
```

Storybook v7 ではこの問題は起きないようで、v6 の場合は注意が必要です。

解決策は、既に Github Issues にも上がっておりました。

[[Bug]: WARN Broken build, fix ... ModuleDependencyError for storybook-start (6/7) · Issue #22159 · storybookjs/storybook](https://github.com/storybookjs/storybook/issues/22159#issuecomment-1538940522)

KANNA では下記のバージョンを指定することで CI エラーも解消されました。

```tsx
"@storybook/jest": "^0.0.10",
"@storybook/test-runner": "^0.9.2",
"@storybook/testing-library": "^0.0.13",
```

CI で回す場合でもバージョン変更が必要になる場合があるので、ここの管理はちゃんとしたほうが良いなと思いました。

### まとめ

以上になります！

特に言語別でテストどう書くか？、というところはネットでも見つけづらかったのでご参考になれたら幸いです。

あと、モジュール依存によるエラーは辛いですね。Storybook v7 では起きないエラーもあるそうなので、早めに上げたいなーと思いました。
