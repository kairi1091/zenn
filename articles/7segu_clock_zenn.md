---
title: "NeoPixelで時間を操る！私たちのオーロラレボリューションクロック"
emoji: "⏰"
type: idea
topics: [ESP32,neopixel,Arduino IDE]
published: true
---
# こんにちは
専門学校の2年生のFGノットです。
今回は授業でクラスのみんなと作ったオーロラレボリューションクロックの私が担当した3つの部分について説明したいと思います。
![](/images/IMG_0451.jpg)

## 配線
neopixelを木の板に張り付けそれを配線していきました。全部で50箇所ぐらいのVCCとGND、信号線を先生にも手伝ってもらいながらはんだ付けしました。とても集中力がいる作業で途中で諦めそうになりましたが最後まで踏ん張って頑張りました。(写真撮り忘れました(´;ω;｀))
## 時間を表示するプログラム
次に、ntpサーバーから受け取った時間をneopixelで表示するプログラムを作成しました。以下がその一部です：
:::details neopixelの表示
```c
#define PIN 27        //INが接続されているピンを指定
#define NUMPIXELS 74  //LEDの数を指定
//プロトタイプ宣言
void ShowTime(int hour, int minute);
const int digitSegments[10][18] = {
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0 },  // 0
  { 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0 },  // 1
  { 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1 },  // 2
  { 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 },  // 3
  { 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1 },  // 4
  { 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 },  // 5
  { 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 },  // 6
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0 },  // 7
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 },  // 8
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 }   // 9
};

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);  //800kHzでNeoPixelを駆動

void ShowTime(int hour, int minute) {
  /* ShowTime関数は、構造型(struct)の変数を受け取れるので、ShowTime関数の中に、LED画面に表示するコードを追加する。*/
  int time_4_digits = hour * 100 + minute;
  //Serial.println(time_4_digits);
  int s[4];
  for (int i = 3; i >= 0; i--) {
    s[i] = time_4_digits % 10;
    time_4_digits = (time_4_digits - s[i]) / 10;
  }

  // Serial.print(s[0]);
  // Serial.print(s[1]);
  // Serial.print(s[2]);
  // Serial.println(s[3]);


  pixels.clear();

  //  int digitSegments1 = digitSegments[s[i]][x] - 1;
  for (int i = 0; i < 4; i++) {     //各ケタ
    for (int j = 0; j < 18; j++) {  //1ケタ分のLED（18個）の表示
      int index = i * 18 + j;

      int huestart = 16363;//0;//始まりの色（書き換える）
      int huefin = 32766;//65536;終わりの色（書き換える)
      int hue = ((huefin - huestart) / 37) * index + huestart;//色の範囲を指定している(ここは書き換えない（0~65535))
      int sat = 255;
      int val = 200;
      
   
      if (i == 2) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));//hue(色),色彩、明るさ
      } else if (i == 3) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
      } else {
        pixels.setPixelColor(index, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
      }
    }
  }
  

  pixels.setPixelColor(36, pixels.Color(flag*100, flag*100, flag*100));//1の時[:]点灯
  pixels.setPixelColor(37, pixels.Color(flag*100, flag*100, flag*100));


  pixels.show();  //LEDに色を反映  
}
```
:::
このコードではひとつのセルのneopixelのLEDの数が18個ありそれを操作して0～9までの数字を表現しています。
そのためにdigitSegmentsというリストを作り0～9までの数字一つ一つに0と1の組み合わせを割り当てています。
最初は0か1ではなく0～18の数字を使って操作しようとしていましたが、余計なLEDが光ったりとバグが出てしまったので0か1にしました。
そのおかげでソース自体を見やすくなり色を変えたりと新しい機能を追加したりできました。
ShowTime関数では時間を表示するには4つの数字で時間を表さないといけないのでfor文を使って一つのセルづつ表示しました。
色を簡単に表せるようにRGBではなくHSVを使って色を表現にしました。
始まりの色と終わりの色を設定することでその色の間だけでグラデーションすることも可能です。

## 時間操作の機能追加
皆さん授業中に時計の進みが遅いと思いませんか？
僕は思います。なのでこの時計に時間操作をできる機能をつけて授業中に操作して授業を早く終わらせてやろうと思いました。
以下がそのプログラムです。
:::details 時間操作
```c
void ClockOperation(){
  if (millis() - waitingtime >= 1000) {   //プログラムが経過した時間が1秒経ったら
    waitingtime = millis();   //基準時間に現在時間を代入
    if (WiFi.status() == WL_CONNECTED) {
      HTTPClient http;

      // URLの設定
      http.begin(serverUrl);

      // GETリクエストの送信
      int httpResponseCode = http.GET();

      if (httpResponseCode > 0) {
      //   // HTTPレスポンスコードを表示
        //Serial.print("HTTP Response code: ");
        //Serial.println(httpResponseCode);

      //   // ペイロードの取得と表示
        String payload = http.getString();
        //Serial.println("Received payload:");
        //Serial.println(payload);

        int dataInt = payload.toInt();
        int time[2];
        
        if (dataInt == 9999){//ntpみにいく
          life = 1;
        }else if(dataInt == 9998){//ntpみにいかない　カウントアップのみ
          life = 2;
          change =0;
        }else{
          for (int i = 1; i >= 0; i--){
            time[i] = dataInt % 100;
            dataInt = (dataInt - time[i]) / 100;
          }
          life = 0;
          timeInfo.tm_hour = time[0];
          timeInfo.tm_min = time[1];

          Serial.print(timeInfo.tm_hour);
          Serial.print(timeInfo.tm_min);
        }
        //Serial.println("Data content copied to another variable:");
        //Serial.print(dataInt);

      } else {
        Serial.print("Error code: ");
        Serial.println(httpResponseCode);
      }

      // リクエストを終了
      http.end();
    } else {
      Serial.println("WiFi Disconnected");
    }
    // 過剰なリクエストを避けるために遅延を追加  // 必要に応じて遅延を調節
  }
  delay(100);
}
```
:::
時間操作をするために新しい関数を作りました。
この関数ではサーバーにアクセスしにいって時間変更してあるとその時間を一桁ずつに分けて変数に格納しています。
時間を見に行った後にntpを見に行くと計画が台無しになってしますのでそこはサーバー側にお願いして時間を変更したら9998に変更してもらうようにしました。
いつも時間操作をしてたいたらばれてしまうのでサーバー側は時間変更しない場合は9999を常に置いといてもらうことにしました。
## これからの展望
これからの展望としてこの7セグクロックは時間機能しかないのでタイマー機能やストップウォッチ、カレンダーなど様々な機能を追加していきたいです。また光り方を時間や時期によって変わるようにしたり、スピーカーをつけたりして音を鳴らしたりしゃべらせたりしたいです。

## チームメンバー
こちらの記事も参考にしてみてください。
https://zenn.dev/ayaponzu2525/articles/seven_segclock
https://zenn.dev/high_machine/articles/7segmentsclock3d