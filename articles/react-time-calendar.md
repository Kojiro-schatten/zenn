---
title: "時間軸のカレンダーを作ってみよう"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react"]
published: false
---

こんにちは！影山です。

KANNA では、以前にカレンダー機能をリリースしました。

[プロジェクト管理アプリ「KANNA」、カレンダー機能をリリース](https://prtimes.jp/main/html/rd/p/000000014.000058603.html)

私も開発に携わっていたのですが、その中でもカレンダー実装は [react-big-calendar](https://github.com/jquense/react-big-calendar) などのライブラリを使わずスクラッチで実装していました。

なかなか大変なプロジェクトだったのですが、得られた経験も大きかったです。

今回は文量が多くなりそうなので、時間軸のカレンダーに絞って共有していきます（余力があれば予定の表示部分も書いていきたい）。

## 挙動の確認

初めに実際に使われている KANNA のカレンダーの挙動を確認していきます。

KANNA の Web 版では

１：自分だけの予定と案件が見れるカレンダー

２：メンバーも含めた予定と案件が見れるカレンダー

の２つが存在しており、それぞれ週と月のカレンダーが用意されています。

下の Gif で確認できるのは、自分だけの予定と案件が見れる週カレンダーです。

![](/images/react-time-calendar/calendar.gif)

Gif 内では80分の予定を作成していますが、表示される予定の縦の長さも実は80(px)になっております。

これは、1時間毎のセルの縦の長さがデザインで60pxになっているため、1分1pxにして実現しました（枠内に収めるために若干スタイルはイジっています）。

また、予定は重複した時は横並びになり、予定と案件を足したイベントの数が４つ以上並ぶと「他N件」という形で表示されます。

他N件をクリックした場合は小モーダルが表示されて、各イベントの情報を確認できます。

今回の記事では、その中でも

１：曜日のロジックについて

２：CSS Gridを使った曜日分割の仕方

３：空きセルクリック時に、そのセルが何日の何時なのかを取得する

まで書いていきます。CodeSandbox を貼っているので、そちらから随時ご確認いただけます。

## 完成系

先に完成形イメージをお見せします。

https://codesandbox.io/s/react-scratch-weekly-time-calendar-d2eb1t?file=/src/styles.css

左側(00:00, 01:00部分)は時間毎に分割しており、右側にまで線が伸びています。

右側の青部分ですが、縦は曜日ごとに、横は先述したように左側から伸びた時間ごとに区切られています。

Chrome の検証ツールからホバーすると、下の画像のように見えることが確認できます。

![](/images/react-time-calendar/calendar-console-view.png)

### １：曜日のロジックについて

曜日は、`WeeklyCalndar.tsx` で扱っています。

```tsx
const weekStartDayOffset = 0;
const _date = dayjs();
const _day = _date.day();
const dayList = Array(7)
  .fill(0)
  .map((_, idx) => {
    const day = weekStartDayOffset + idx;
    const dayFormat = dayjs(
      _date.date(_date.date() - _day + weekStartDayOffset + idx)
    );
    return { day: day, date: dayFormat.format("YYYY-MM-DD") };
  });
```

`WeeklyCalendar.tsx` では1週間分の日付を作成して、曜日（この記事では使っていないですが）も番号で管理しています。

`weekStartDayOffset` は曜日の始まりを設定しており、今回であれば0(日曜)始まりにしています。月曜始まりにしたい場合、 `weekStartDayOffset` を1にすることで曜日始まりを変えられます。

### ２：Gridを使った曜日分割の仕方

次に、CSS Grid で縦分割するようにしてみました。

`Timeline.tsx` では、左側に `timeslotWrapper` (00:00, 01:00 の部分)、右側に `timelineWrapper` (真白な部分)と分けております。

この `timelineWrapper` 内になる `eventContainer` の CSS は

```css
grid-template-columns: repeat(7, 1fr);
```

と指定しており、 `grid-template-columns` を使って縦に7分割して曜日毎に区切れる形にしています。

### ３：空きセルクリック時に、そのセルが何日の何時なのかを取得する

CodeSandbox 内で各セルをクリックすると、その日付と時間がログ出力されるようになっています。

セルクリック時の流れとしては、

１：`eventContainer` のなかで、dayList(曜日)毎に map する

```tsx
// Timeline.tsx l.54~
<div className="eventContainer">
  {dayList.map((dayItem, index) => {
    return (
      <div
        key={dayItem.date}
        style={{ gridColumn: index + 1 }}
        className="calendarColumn"
      >
        <div className="date">
          {dayItem.date.split("-").at(1)}-
          {dayItem.date.split("-").at(2)}
        </div>
        <EmptyCell date={dayItem.date} />
      </div>
    );
  })}
</div>
```

２：曜日内で `EmptyCell` コンポーネントを作り、date を props として渡す。

３： `EmptyCell` では、時間(HOUR_LIST)毎に map をして、24個のセルコンポーネントを作成(1個のセル1時間に対応しているため)

４：props で渡ってきた曜日情報と、 hourlist で定義されている hour を使い各曜日と時間を表示する

```tsx
// Timeline.tsx l.12~
const EmptyCell = (date: { date: string }) => {
  return (
    <>
      {HOUR_LIST.map((hourList, index) => {
        return (
          <>
            <div
              onClick={() => {
                console.log(date, `${hourList.hour}時`);
              }}
              className="empty"
            />
          </>
        );
      })}
    </>
  );
};

{dayList.map((dayItem, index) => {
  return (
    <div
      style={{ gridColumn: index + 1 }}
      className="calendarColumn"
    >
      <div className="date">
        {dayItem.date.split("-").at(1)}-
        {dayItem.date.split("-").at(2)}
      </div>
      <EmptyCell date={dayItem.date} />
      <div>{/* ここに実際の予定(イベント)を表示するコンポーネントが入る */}</div>
    </div>
  );
})}
```

といった感じです。

### まとめ

以上が、時間軸カレンダーを作る際の流れとなります！参考になったら幸いです。

時間のマスを定義したり、クリックイベントでどうやってその曜日を取得するか が難しかったりしますが、曜日ごとに map してから作ると対応しやすいことが分かりました。

CSS Grid を上手に使えばもっと楽な実装になるかと思えそうですが、仕様とも相まってガッツリ使うことはできなかったのが少し悔しいですが、まぁ動いているのでヨシ！。
