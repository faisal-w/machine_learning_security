###### 編集中

## 2.0. 目的
本ブログは、**スパム検知システムの実装**を通して、**ナイーブベイズ**と呼ばれる**テキスト分類アルゴリズム**の使用方法を理解することを目的とします。  

## 2.1. スパム検知
スパム検知（Spam Detection）とは、受信者の意向を無視して無差別かつ大量にばら撒かれるスパムメールを検知する機能であり、メジャーなメーラーには標準で実装されています。  

### 2.1.1. コンセプト
ところで、スパム検知を実装するためには、受信メールのタイトルや本文からスパムメールの特徴を抽出し、これを手掛かりにメールを正常またはスパムに分類すれば良いと考えられます。また、可能であれば、分類時にそのパーセンテージ（確率）も併せて計算できた方が運用上は好ましいと言えます。なぜならば、メールには機械的に白黒つけ難い微妙なメールが存在するからです。このような場合は、例えば「分類確率が70%以下の場合は人間の判断を仰ぐ」等のようなルールを決めておくことで、誤検知を減らすことができます。  

本ブログではナイーブベイズを使用し、上記のコンセプトを持ったスパム検知システムを実装していきます。  

## 2.2. ナイーブベイズ（Naive Bayes）
ナイーブベイズは**テキスト分類**を行う機械学習アルゴリズムであり、分類対象のテキストを予め定義した**カテゴリ**（=クラス）に分類することができます。ナイーブベイズはスパム検知の他に、ニュース記事やブログのジャンル分け（政治、スポーツ、科学技術、エンタメ等）、[Webページの種別判定](https://www.mbsd.jp/blog/20160113.html)等に使用されています。  

### 2.2.1. ナイーブベイズ入門
いきなり質問ですが、あなたなら以下のテキストを「ラーメン」or「脳神経学」のどちらのカテゴリに分類するでしょうか？  

```
私が以下の画像を見たとき、瞬時に「味噌ラーメン」であると認識することができる。
私の脳内では、目から入力された画像(入力信号)が最初のニューロンに伝わり、
幾つかのニューロンが発火しながら画像情報を伝播・分類していき、
最終的に「これは味噌ラーメンです。」と判断を下している。
※ちなみに、この画像は「○○軒」の白味噌ラーメン。
```

私の場合は「脳神経学」に分類します（それは正解です）。  
これは、私が**過去に見聞き（学習）した知識**を基に、テキストに含まれる「`認識`、`脳内`、`信号`、`ニューロン`、`発火`、`伝播`、`判断`」等の単語から「脳神経学！」と判断したからです。ところで、このテキストにはラーメンに関する単語も含まれています。例えば「`味噌`、`ラーメン`、`白味噌`、`軒`」等です。ではなぜ私は「ラーメン！」と判断しなかったのでしょうか。それは、このテキストにおいては、ラーメンに関する単語よりも**脳神経学に関する単語の方が出現頻度が高い**からです。つまり、（単一の単語のみを見るだけではなく）テキストを構成する全単語を**総合的に見る**ことで正しくカテゴリ分類ができたわけです。この時、テキストの文脈は考慮しません。  

### 2.2.2. 処理の流れ
ナイーブベイズも上記と似たようなロジックでテキスト分類を行います。  
つまり、**テキストの文脈は考慮せず**、**単語の出現頻度**を基に分類します（このようなモデルを**Bag-of-words**と呼びます）。  

| Bag-of-wordsモデル（BoW）|
|:--------------------------|
| 自然言語処理で使用される文書の単純化表現。文書の構文（文法や単語の順番）は考慮せず、どのような単語が含まれているかのみを考慮する。1つの袋に様々な単語がごちゃ混ぜに入っているイメージ。|

ナイーブベイズは教師あり学習モデルです。よって、上述の「**過去に見聞き（学習）した知識**」に相当する学習データを準備する必要がありますが、ナイーブベイズでは学習データを**カテゴリテーブル**と呼ばれるデータセットで表現します。  

以下は、上記テキストを分類するためのカテゴリテーブル例です。  

| カテゴリ | カテゴリを構成する単語 |
|:--------|:------------------|
| 脳神経学 | 認識、脳内、目、入力信号、ニューロン、伝、発火、伝播、分類、判断 ...|
| ラーメン | 味噌、ラーメン、白味噌、軒、濃厚、マイルド、味、美味、こってり、バリカタ ...|

「カテゴリ」と「各カテゴリを構成する単語」を紐付けただけの非常にシンプルな構造です。  
このカテゴリテーブルを使用し、各カテゴリに紐付く単語が分類対象のテキストに幾つ含まれてるのかカウントして単語の出現頻度を計算します。  

そのためには、テキスト内の文書を単語毎に分割する必要があります。  
ナイーブベイズでテキスト分類する際は、事前に分類対象のテキストを[**NLTK（Natural Language Toolkit）**](https://www.nltk.org/)等で**単語分割**し、各単語をテキストから切り出します。  

| NLTK（Natural Language Toolkit）|
|:--------------------------|
| 自然言語（人間が日常的に使用している言語）に対して様々な処理を行うことができるPython用のライブラリ。|

| 単語分割|
|:--------------------------|
| 日本語や中国語などのように、単語間に区切りが存在しない膠着語において、単語の間に明示的な区切りを入れる**自然言語処理**の手法。例えば、「`これは味噌ラーメンです。`」をNLTKで単語分割すると「`/これは/味噌/ラーメン/です/。/`」となる（"`/`"が区切り文字）。なお、英語は単語間に空白が存在するため、容易に単語分割が可能。|

| 自然言語処理（Natural Language Processing）|
|:--------------------------|
| 人間が日常的に使用している自然言語をコンピュータで処理する技術群。|

以下は、NLTKを使用して単語分割を行った結果を示しています（区切り文字は"`/`"）。  

```
私/が/以下/の/画像/を/見/たとき/、/瞬時/に/「/味噌/ラーメン/」/であると/認識/することができる/。
/私/の/脳内/では/、/目/から/入力/された/画像/(/入力信号/)/が/最初/の/ニューロン/に/伝/わり/、
/幾/つかの/ニューロン/が/発火/しながら/画像情報/を/伝播/・/分類/していき/、
/最終的/に/「/これは/味噌/ラーメン/です/。」/と/判断/を/下/している/。
/※/ちなみに/、/この/画像/は/「○○/軒/」/の/白味噌/ラーメン/。
```

次節では、ナイーブベイズを使用したスパム検知システムの構築手順と実装コードを解説します。  

## 2.3. スパム検知システムの実装
ナイーブベイズを使用し、受信したメールがスパムなのか正常なのかを検知するシステムを構築します。  

### 2.3.1. 学習データの準備
ナイーブベイズは教師あり学習のため、事前にスパムや正常メールを含んだ学習データを準備する必要があります。  
本来であれば、筆者が受信したスパムメールを使用することが好ましいですが、不幸なことにサンプル数が少ないため、スパムメール事例を掲載した下記サイトからスパムメールを収集します。  

 * [最近の迷惑メール例](http://zero-net.jugem.jp/?eid=791)  
 * [つい読んでしまいたくなる？本当にあった迷惑（スパム）メール例集](https://securitynavi.jp/1081)  

これらのサイトには様々なスパムメールの事例が掲載されていることから、スパム検知システムの学習データとして使用することにしました。  
なお、正常メールについては、筆者が信頼できる相手から受信したメールをサンプリングします。  

次に、スパムメールと正常メールを特徴付ける**単語を選別**します。  
後述しますが、本ブログではメール本文から**単語を切り出す**ために[**NLTK（Natural Language Toolkit）**](https://www.nltk.org/)を使用し、収集したスパムメールと正常メールの本文を**単語分割**します。  

| NLTK（Natural Language Toolkit）|
|:--------------------------|
| 自然言語（人間が日常的に使用している言語）に対して様々な処理を行うことができるPython用のライブラリ。|

| 単語分割|
|:--------------------------|
| 日本語や中国語などのように、単語間に区切りが存在しない膠着語において、単語の間に明示的な区切りを入れる**自然言語処理**の手法。例えば、「`これは最終通告です。`」を単語分割すると「`これは/最終通告/です/。`」となる（"|"が区切り文字）。なお、英語は単語間に空白が存在するため、容易に単語分割が可能。  

| 自然言語処理（Natural Language Processing）|
|:--------------------------|
| 人間が日常的に使用している自然言語をコンピュータで処理する技術群。|

分割された単語の中から、スパムメールおよび正常メールを特徴付けると思われる単語を選別し、それぞれ**Spam**/**Normal**というようにラベル付けを行います（ラベル名は任意）。すると、以下のようなファイル内容になります。  

```
有料,Spam
無料,Spam
料金,Spam
特典付,Spam
懸賞付,Spam
登録料金,Spam
利用料金,Spam
料金未払,Spam
未払い金,Spam
支払い,Spam
最終通告,Spam
本通知,Spam
連絡,Spam
配信停止,Spam
再開,Spam

...snip...

確認,Normal
修正,Normal
アンケート,Normal
機能,Normal
結果,Normal
質問事項,Normal
診断日程延期,Normal
相談,Normal
連絡,Normal
メディア,Normal
記事掲載,Normal
本件,Normal
当社,Normal
Web,Normal
サイト,Normal
```

1カラム目は特徴量、2カラム目がラベル（メールの種別＝答え）を表しています。  
この状態のファイルを学習データ「[train_spam.csv](https://github.com/13o-bbr-bbq/machine_learning_security/blob/master/Security_and_MachineLearning/dataset/train_spam.csv)」とし、**UTF-8**で保存しておきます。  

### 2.3.2. テストデータの準備
スパム検知システムの性能を評価するためのテストデータを準備します。  
ここでは、自分がスパムメールの送信者になったつもりで、自由に怪しげなメールを作成します。  

 * スパムメール例  
 ```
 件名：警告
 これは最終通告です。
 5分以内に以下のサイトにアクセスして、退会手続を行ってください。
 http://www.example.com

 退会しない場合は、即刻法的手続を行います。
 ```

「最終通告」や「法的手続」等の恐怖を煽る単語を複数配置し、URLを付与しました。  

 * 正常メール例  
 ```
 件名：ログイン情報
 このたびは会員登録いただき、誠にありがとうございます。
 登録が完了しましたので、ログイン情報をご連絡いたします。
 ※ご注意（以下を必ずお読みください）
 ログイン情報-----------------------
 ログインURL：https://www.example.com
 ログインID：cysec.ml.train@gmail.com
 ログインパスワード：password'
 ```

このメールは全体的に柔らかい単語を使用しています。  
ちなみに、本メールは筆者がとあるサービスで会員登録した際に実際に受信したメールを基にしています（もちろんスパムメールではありません）。  

これで、学習データとテストデータの準備が完了しました。  
次項では実際にサンプルコードを実行し、実際にスパムメールと正常メールを正しく判別できるのか検証します。  

### 2.3.3. サンプルコード及び実行結果
#### 2.3.3.1. サンプルコード
本ブログではPython3を使用し、簡易的なスパム検知システムを実装しました。  
※本コードは[こちら](https://github.com/13o-bbr-bbq/machine_learning_security/blob/master/Security_and_MachineLearning/src/spam_detection.py)から入手できます。  

本システムの大まかな処理フローは以下のとおりです。  

 1. 学習データのロード  
 2. ナイーブベイズモデルの作成  
 3. 受診メールの取得  
 4. 受信メールのスパム判定  

```
#!/bin/env python
# -*- coding: utf-8 -*-
import codecs
import urllib.request
import nltk
import mailbox
import numpy as np
import pandas as pd
from email.header import decode_header
from nltk.corpus.reader import *
from nltk.corpus.reader.util import *
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer

# Max target mail number.
MAX_COUNT = 5


# Get stopwords (Japanese and symbols).
def get_stopwords():
    url = 'http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt'
    slothlib_file = urllib.request.urlopen(url)
    slothlib_stopwords = [line.decode('utf-8').strip() for line in slothlib_file]
    jpn_stopwords = [ss for ss in slothlib_stopwords if not ss == u'']
    jpn_symbols = ['’', '”', '‘', '。', '、', 'ー', '！', '？', '：', '；', '（', '）', '＊', '￥']
    eng_symbols = ["'", '"', '`', '.', ',', '-', '!', '?', ':', ';', '(', ')', '*', '--', '\\']
    return jpn_stopwords + jpn_symbols + eng_symbols


# Judge Spam.
def judge_spam(X_train, y_train, stop_words, message, jp_sent_tokenizer, jp_chartype_tokenizer):
    fout = codecs.open('..\\dataset\\temp.txt', 'w', 'utf-8')
    fout.write(message)
    fout.close()
    spam = PlaintextCorpusReader('..\\dataset\\', r'temp.txt', encoding='utf-8',
                                 para_block_reader=read_line_block,
                                 sent_tokenizer=jp_sent_tokenizer,
                                 word_tokenizer=jp_chartype_tokenizer)

    # Tokenize.
    tokenize_message = '|'.join(spam.words())
    lst_bow = tokenize_message.replace('\n', '').replace('\r', '').split('|')
    while lst_bow.count('') > 0:
        lst_bow.remove('')

    # Remove stop words.
    vectorizer = CountVectorizer(stop_words=stop_words)

    # Fit (Train).
    vectorizer.fit(X_train)
    X_train = vectorizer.transform(X_train)
    clf = MultinomialNB(alpha=1.0)
    clf.fit(X_train, y_train)

    # Test.
    X_test = vectorizer.transform(lst_bow)
    y_preds = clf.predict_proba(X_test)

    # Classify using Naive Bayes (MultinomialNB).
    result = ''
    if np.sum(y_preds[:, 0]) > np.sum(y_preds[:, 1]):
        result = clf.classes_[0]
    else:
        result = clf.classes_[1]

    print('[Judgement]\nThis mail is <{}>.\n{}: {}%, {}: {}%\n'.format(result,
                                                                       clf.classes_[0],
                                                                       str(round(np.mean(y_preds[:, 0])*100, 2)),
                                                                       clf.classes_[1],
                                                                       str(round(np.mean(y_preds[:, 1])*100, 2))))


if __name__ == '__main__':
    # Load train data.
    df_data = pd.read_csv('..\\dataset\\train_spam.csv', header=None)
    X = [i[0] for i in df_data.iloc[:, [0]].values.tolist()]
    y = [i[0] for i in df_data.iloc[:, [1]].values.tolist()]

    # Setting NLTK
    jp_sent_tokenizer = nltk.RegexpTokenizer(u'[^　「」！？。]*[！？。]')
    jp_chartype_tokenizer = nltk.RegexpTokenizer(u'([ぁ-んー]'
                                                 u'+|[ァ-ンー]'
                                                 u'+|[\u4e00-\u9FFF]'
                                                 u'+|[^ぁ-んァ-ンー\u4e00-\u9FFF]+)')

    # Get stop words.
    stop_words = get_stopwords()

    # Get inbox data (ex. Mozilla Thunderbird).
    mail_box = mailbox.mbox('Your mailbox path')
    for idx, key in enumerate(mail_box.keys()[::-1]):
        if idx + 1 > MAX_COUNT:
            break

        # Get message data and receive date.
        msg_item = mail_box.get(key)
        recv_date = msg_item.get_from()

        # Get subject from message data.
        subject = ''
        for msg_subject, enc in decode_header(msg_item['Subject']):
            if enc is None:
                subject += msg_subject
            else:
                subject += msg_subject.decode(enc, 'ignore')

        # Get message body from message data.
        content = ''
        for msg_body in msg_item.walk():
            if 'text' not in msg_body.get_content_type():
                continue
            if msg_body.get_content_charset():
                content = msg_body.get_payload(decode=True).decode(msg_body.get_content_charset(), 'ignore')
            else:
                if 'charset=shift_jis' in str(msg_body.get_payload(decode=True)):
                    content = msg_body.get_payload(decode=True).decode('cp932', 'ignore')
                else:
                    print('Cannot decode : message key={}.'.format(key))
                continue

        # Judge Spam.
        if subject != '' or content != '':
            print('-'*50)
            print('[Received date]\n{}\n\n[Message]\n件名: {}\n{}'.format(recv_date, subject, content))
            judge_spam(X, y, stop_words, subject + '。' + content, jp_sent_tokenizer, jp_chartype_tokenizer)
        else:
            print('This message is empty : message key={}.'.format(key))
```

このプログラムは、Thunderbirdのような電子メールクライアントのメールボックスから最新5件（`MAX_COUNT`）のメールを取得してスパム判定します。  

#### 2.3.3.2. コード解説
今回はナイーブベイズの実装に、機械学習ライブラリの**scikit-learn**を使用しました。  
※scikit-learnの使用方法は[公式ドキュメント](http://scikit-learn.org/)を参照のこと。  

##### パッケージのインポート
```
import nltk
...snip...
from nltk.corpus.reader import *
from nltk.corpus.reader.util import *
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer
```

scikit-learnのナイーブベイズ用パッケージ「`MultinomialNB`」をインポートします。  
このパッケージには、ナイーブベイズでテキスト分類を行うための様々なクラスが収録されています。  

また、単語分割や**ストップワード**の除去を行うために、NLTKのパッケージ「`nltk`」もインポートします。  

| ストップワード（Stop Word）|
|:--------------------------|
| 自然言語処理の精度向上のため、処理対象から除外せざるを得ない単語。日本語では「ここ」「は」「はい」「これら」など、英語では「is」「a」「this」「the」等のありふれた、**テキストの特徴付けに貢献しない単語や記号**等。|

##### 学習データのロード
```
# Load train data.
df_data = pd.read_csv('..\\dataset\\train_spam.csv', header=None)
X = [i[0] for i in df_data.iloc[:, [0]].values.tolist()]
y = [i[0] for i in df_data.iloc[:, [1]].values.tolist()]
```

学習データ「[train_spam.csv](https://github.com/13o-bbr-bbq/machine_learning_security/blob/master/Security_and_MachineLearning/dataset/train_spam.csv)」から、各特徴量とラベルを取得します（特徴量：`X`、ラベル：`y`）。  

##### 単語分割の事前準備
```
# Setting NLTK
jp_sent_tokenizer = nltk.RegexpTokenizer(u'[^　「」！？。]*[！？。]')
jp_chartype_tokenizer = nltk.RegexpTokenizer(u'([ぁ-んー]'
                                             u'+|[ァ-ンー]'
                                             u'+|[\u4e00-\u9FFF]'
                                             u'+|[^ぁ-んァ-ンー\u4e00-\u9FFF]+)')
```

「。」「！」「？」で終わる文字列を1つの文として認識して分割するために、`nltk`の正規表現パターンを定義します（`jp_sent_tokenizer`）。  
また、文字種を検出するために、unicodeの範囲（平仮名は [ぁ-ん] 、カタカナは [ァ-ン] 、漢字は[U+4E00 ～ U+9FFF]）を定義します（jp_chartype_tokenizer）。  

##### ストップワードの定義
```
# Get stopwords (Japanese and symbols).
def get_stopwords():
    url = 'http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt'
    slothlib_file = urllib.request.urlopen(url)
    slothlib_stopwords = [line.decode("utf-8").strip() for line in slothlib_file]
    jpn_stopwords = [ss for ss in slothlib_stopwords if not ss == u'']
    jpn_symbols = ['’', '”', '‘', '。', '、', 'ー', '！', '？', '：', '；', '（', '）', '＊', '￥']
    eng_symbols = ["'", '"', '`', '.', ',', '-', '!', '?', ':', ';', '(', ')', '*', '--', '\\']
    return jpn_stopwords + jpn_symbols + eng_symbols

...snip...

# Get stop words.
stop_words = get_stopwords()
```

ストップワードを定義します。  
今回のストップワード群は、日本語ストップワードと記号（全角/半角）を組み合わせて作成します。  
日本語のストップワードは、一般に公開されている[SlothLib](http://svn.sourceforge.jp/svnroot/slothlib/CSharp/Version1/SlothLib/NLP/Filter/StopWord/word/Japanese.txt)を使用します（`jpn_stopwords`）。また、記号のストップワードは、筆者がスパム検知には貢献しないと考えた幾つかの記号を定義します（`jpn_symbols`、`eng_symbols`）。  

これらのワードや記号をリスト化して、ストップワードリストを作成します。  

##### ストップワードの削除
```
# Remove stop words.
vectorizer = CountVectorizer(stop_words=stop_words)
```

単語をベクトル化する際に使用する`CountVectorizer`の引数`stop_words=`」にストップワードリストを指定することで、単語をベクトル化するタイミングで学習データやテストデータ（受信メール）に含まれるストップワードを除去することができます。  

##### モデルの作成
```
# Fit (Train).
vectorizer.fit(X_train)
X_train = vectorizer.transform(X_train)
clf = MultinomialNB(alpha=1.0)
```

学習データに含まれる単語をベクトル化します（`vectorizer.fit(X_train)`、`vectorizer.transform(X_train)`）。
その後、`MultinomialNB`でナイーブベイズモデルを作成します。  
`MultinomialNB`の引数として渡している「`alpha=1.0`」は、**ラプラススムージング**で加算する値を指定します。  

| ラプラススムージング（Laplace Smoothing）|
|:--------------------------|
| **ゼロ頻度問題**を回避するための手法。各カテゴリの単語出現率を計算する際、テストデータに含まれる単語がカテゴリに1つも存在しない場合、単語の出現回数に任意の数値（通常は1が多い）を加えて単語出現率が0になるのを防ぐ。|

| ゼロ頻度問題（Zero Frequency Problem）|
|:--------------------------|
| テストデータに含まれる単語がカテゴリに1つも存在しない場合、単語の出現率が0になる問題。単語出現率が0になった場合、テキスト分類の精度が悪くなる。|

##### 学習の実行
```
clf.fit(X_train, y_train)
```

作成したモデル`clf`の`fit`の引数として、特徴量「`X_train`」とラベル「`y_train`」を渡すことで、モデルの学習が行われます。  

##### 分類結果の取得
```
# Test.
X_test = vectorizer.transform(lst_bow)
y_preds = clf.predict_proba(X_test)
```

受信したテストデータに含まれる単語をベクトル化します（`X_test = vectorizer.transform(lst_bow)`）。  
ここで、引数の`lst_bow`は、受信したメール本文を単語分割し、ストップワードを除去した単語のリストです。  

学習済みモデル`clf`の`predict_proba`の引数として、ベクトル化したテストデータ「`X_test`」を渡すことで、テストデータに含まれる各単語の分類確率を得る事ができます。  

##### 分類結果の出力
```
result = ''
if np.sum(y_preds[:, 0]) > np.sum(y_preds[:, 1]):
    result = clf.classes_[0]
else:
    result = clf.classes_[1]
```

リストとして保存されている各単語の分類確率「`y_preds`」から、各カテゴリ（スパム or 正常）のスコア（各単語の分類確率の和）を比較し、スパムメールまたは正常メールであるのかを判定します。  

##### 分類確率の出力
```
print('[Judgement]\nThis mail is <{}>.\n{}: {}%, {}: {}%\n'.format(result,
                                                                   clf.classes_[0],
                                                                   str(round(np.mean(y_preds[:, 0])*100, 2)),
                                                                   clf.classes_[1],
                                                                   str(round(np.mean(y_preds[:, 1])*100, 2))))
```

リストとして保存されている各単語の分類確率「`y_preds`」から、各カテゴリ（スパム or 正常）の分類確率（各単語の分類確率の平均）と判定結果をコンソールに出力します。  

#### 2.3.3.3. 実行結果
上記で作成したテストデータ（スパムメールまたは正常メール）を実際に送信し、スパム判定ができるのか確認します。  

1通目のメールは以下のとおりです。  
これは**スパムメール**を想定して用意したテストデータです。  

![スパムメール](https://github.com/13o-bbr-bbq/machine_learning_security/blob/master/Security_and_MachineLearning/img/spam_mail.png)

これに対するスパム検知システムの出力結果は以下のとおりです。  

 ```
 [Received date]
 - Thu Sat 29 15:14:18 2018

 [Message]
 件名: 警告
 これは最終通告です。
 5分以内に以下のサイトにアクセスして、退会手続を行ってください。
 http://www.example.com

 退会しない場合は、即刻法的手続を行います。

 [Judgement]
 This mail is <Spam>.
 Normal: 46.85%, Spam: 53.15%
 ```

各カテゴリの分類確率を見ると、カテゴリ「`Normal`」は**46.85%**、カテゴリ「`Spam`」は**53.15**%であることが分かります。よって、カテゴリ「`Spam`」の確率が上回っているため、1通目のメールは**スパムメール**であると判定されます（`This text is <Spam>`）。  

2通目のメールは以下のとおりです。  
これは**正常メール**を想定して用意したテストデータです。  

![正常メール](https://github.com/13o-bbr-bbq/machine_learning_security/blob/master/Security_and_MachineLearning/img/normal_mail.png)

これに対するスパム検知システムの出力結果は以下のとおりです。  

 ```
 [Received date]
 - Thu Sat 29 15:16:12 2018

 [Message]
 件名: ログイン情報
 このたびは会員登録いただき、誠にありがとうございます。
 登録が完了しましたので、ログイン情報をご連絡いたします。
 ※ご注意（以下を必ずお読みください）
 ログイン情報-----------------------
 ログインURL：https://www.example.com
 ログインID：cysec.ml.train@gmail.com
 ログインパスワード：password'

 [Judgement]
 This mail is <Normal>.
 Normal: 50.4%, Spam: 49.6%
 ```

各カテゴリの分類確率を見ると、カテゴリ「`Normal`」は**50.4%**、カテゴリ「`Spam`」は**49.6**%であることが分かります。よって、カテゴリ「`Normal`」の確率が上回っているため、2通目のメールは**正常メール**であると判定されます（`This text is <Normal>`）。  

## 2.4. おわりに
このように、簡易的な実装であるものの、ナイーブベイズを使うことで受信メールのスパム判定ができることが分かりました。  
本ブログではスパム判定を例にしましたが、Webクローラーの**ページ種別判定エンジン**（詳細は[こちら](https://www.mbsd.jp/blog/20160113.html)）、ニュース記事やブログのジャンル分類、WAFの攻撃検知等に利用することもできます。  

ナイーブベイズは手軽に実装ができ、かつ利用用途も広いため、ご興味を持ちましたら様々なタスクに利用してみる事をお勧めします。  

## 2.5. 動作条件
 * Python 3.6.1（Anaconda3）
 * nltk 3.3.0
 * numpy 1.15.1
 * scikit-learn 0.20.0
