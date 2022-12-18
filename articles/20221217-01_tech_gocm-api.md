---
title: "Goでオレオレ証明書を管理してみる"
emoji: "🗝"
type: "tech"
topics: ["Go", "証明書"]
published: true
---
本記事は [Go Advent Calendar 2022](https://qiita.com/advent-calendar/2022/go) - カレンダー1 17日目の記事になります。

Advent Calendarの参加も3年目になりました。
今年もコードの準備がギリギリでしたよ、ええ（恒例）。

# はじめに
2022年の初めから基幹システムをAWSへ移行する大規模プロジェクトに携わっていました。
AWSを勉強していく中でAWS Certificate Manager(ACM)というサービスが個人的に気になりました。

https://aws.amazon.com/jp/certificate-manager/

簡単に言ってしまいますと、証明書の管理もしてくれて、且つAWSのマネージドサービスへ証明書を素早く展開してくれるマネージドサービスとなっております。
発行した証明書をEC2等のWEBサーバ内にインストールすることももちろんできます。

なぜ気になったのかと申しますと、単純に「我が家にも欲しい！！」という欲求です。
我が家ではオレオレ証明書を複数枚発行しておりまして、管理が煩雑になっておりました。
証明書はファイルに保存するものという固定概念に囚われていたため、そうなっていたのでした。

ACMのサービスを見て、今までファイルで管理していた証明書をDBで管理し、REST APIで取得できるようにすればLet's Encryptのような仕組みが可能なのではないかと考えました。

# 筆者環境
* OS: Arch Linux
* Go: 1.19.4 linux/amd64
* Editor: code-server 4.9.1 / vim 9.0
* パッケージ管理: Go Modules
* サービス管理: systemd 252 (252.3-1-arch)
* ビルドツール: Gnu Make

今回は使用しませんでしたが、Goにも遂にジェネリクスが導入されましたね。
C#でジェネリクスを学んだ身なので違いに戸惑うことは多いですが、できるようになったことが多くなった点は素直に嬉しいです。
これからも引き続きGoを愛用していこうと思います。

# 注意事項
今回はセキュリティに関するシステムを制作しました。
当然脆弱性等に気をつけながら設計・実装はしておりますが、個人が使うツールとしてある程度手抜きをしている部分もございます。
当ツールは基本的に **ローカル環境** で使用することを前提に制作しておりますので、インターネット環境で使用することは考えておりません。

当ツールを利用・改変する際はMIT Licenseのライセンス条項に則って、ご自身でコードの確認の上ご使用ください。

# 自作か既製品の流用か
今回のツール制作に当たり事前に既製品がないか調べてみました。
すると、 **step-ca** というOSSが見つかりました。

https://smallstep.com/certificates/

ただ、ACMのように簡単なAPIだけで証明書が発行できるような代物ではなさそうです。
ACMEプロトコルに対応しているOSSということはすごいと思ったのですが、わざわざローカルでACMEの手順を使って証明書発行するのもどうなの？と感じ自作することにしました。
（ローカル環境でわざわざcertbotとか使いたくないです。）

# 証明書発行ってどうするの？
一般的にオレオレ証明書の発行をしよう！と思うとopensslで発行することになると思います。
opensslでの発行を自動化したいと思いツールを作ったことがあります。

https://gist.github.com/t-okkn/658d78245e9758f7cf69e6ef316d52e2

しかし、証明書ストアは結局フォルダの中になり管理が煩雑になります。
なのでGoで発行してDBにpemデータを格納させようと考えました。

まず行き詰まった点は、プログラム言語で証明書が発行できるのか？ということです。
C#に [`X509Certificate2`](https://learn.microsoft.com/ja-jp/dotnet/api/system.security.cryptography.x509certificates.x509certificate2?view=net-6.0) というクラスがあり、証明書を読み込むことができることは知っていました。
ということはGoにも `crypto` の中にPrivateKey発行のためのパッケージや証明書が発行できる `x509` みたいなパッケージがあるのかと思い漁ってみると、ありました。

https://pkg.go.dev/crypto/x509

`CreateCertificate` なるAPIも存在しているので、証明書が作成できそうだということもすぐにわかりました。
ただ、あることはわかったのですが、APIドキュメントを眺めてみてもイマイチわかりません。
証明書の発行手順はわかっていますが、自己署名する場合は引数に何を入れるのか？とか。

そんなときググっていると神記事を発見。

https://qiita.com/tardevnull/items/1c81df51e5b7b5fb4243

私が知りたかったことの全てが記載されていました。
マジ感謝です😭

# 作成物
仕組み自体はそれほど難しいものではありません。

![簡単な仕組み図](https://storage.googleapis.com/zenn-user-upload/680d169fe3ea-20221217.png)

1. REST APIによって証明書を発行する
2. 発行された証明書と暗号化した秘密鍵をDBに格納
3. 格納されている証明書をREST APIによって取得

という仕組みです。

https://github.com/t-okkn/gocm-api

# 証明書発行手順

## １．秘密鍵作成
まずは証明書発行に欠かせない秘密鍵を作成します。
秘密鍵のアルゴリズムはRSA（2048bit, 4096bit）、ECDSA（P-256, P-384, P-521）、ED25519に対応しています。

```go:cert/privatekey.go
type PrivateKeyAlgorithm string

type PrivateKey struct {
	Algorithm PrivateKeyAlgorithm
	Key       crypto.Signer
}

const (
	UNKNOWN_ALGORITHM PrivateKeyAlgorithm = "UNKNOWN"
	RSA               PrivateKeyAlgorithm = "RSA"
	ECDSA             PrivateKeyAlgorithm = "ECDSA"
	ED25519           PrivateKeyAlgorithm = "ED25519"
)

// RSA PrivateKeyを生成します
//
// ※2048bit, 4096bitにのみ対応しています
func GenerateRSAKey(bits int) (PrivateKey, error) {
	if bits != 2048 && bits != 4096 {
		e := fmt.Sprintf("指定したビット数（%dbit）には対応していません", bits)
		return PrivateKey{}, errors.New(e)
	}

	priv, err := rsa.GenerateKey(rand.Reader, bits)

	if err != nil {
		return PrivateKey{}, err
	}

	k := PrivateKey{
		Algorithm: RSA,
		Key:       priv,
	}

	return k, nil
}

// ECDSA PrivateKeyを生成します
//
// ※P-256, P-384, P-521にのみ対応しています
func GenerateECDSAKey(bits int) (PrivateKey, error) {
	var curve elliptic.Curve

	switch bits {
	case 256:
		curve = elliptic.P256()

	case 384:
		curve = elliptic.P384()

	case 521:
		curve = elliptic.P521()

	default:
		e := fmt.Sprintf("指定したビット数（%dbit）には対応していません", bits)
		return PrivateKey{}, errors.New(e)
	}

	priv, err := ecdsa.GenerateKey(curve, rand.Reader)

	if err != nil {
		return PrivateKey{}, err
	}

	k := PrivateKey{
		Algorithm: ECDSA,
		Key:       priv,
	}

	return k, nil
}

//ED25519 PrivateKeyを生成します
func GenerateED25519Key() (PrivateKey, error) {
	_, priv, err := ed25519.GenerateKey(rand.Reader)

	if err != nil {
		return PrivateKey{}, err
	}

	k := PrivateKey{
		Algorithm: ED25519,
		Key:       priv,
	}

	return k, nil
}
```

秘密鍵の生成については型の違い（アルゴリズムの違い）を吸収するために `PrivateKey` 構造体を用意しています。
秘密鍵の型がすべて収まるinterfaceとして `crypto.Signer` が用意されてありましたので、それを使用しました。

## ２．証明書発行
opensslで証明書を発行したことがある方なら必ずこう思うはずです。
「CSRの生成はどこにいった？」と。
どうやらGoでは `x509.Certificate` 構造体に情報を詰め込んで、 `CreateCertificate` を実行してあげるとちゃんとした証明書になるようです。

私も半信半疑だったので、検証段階でGoで生成した証明書をopensslで確認をしてみましたが、何も問題はありませんでした。

```go:cert/core.go
type CertType string

type CertData struct {
	CAID           string
	Serial         uint32
	CommonName     string
	PrivateKey     PrivateKey
	Type           CertType
	PemData        string
	Created        string
	ExpirationDate string
}

type CreateCACertRequest struct {
	CAID           string
	PrivateKey     PrivateKey
	Subject        pkix.Name
	Serial         uint32
}

const (
	DT_FORMAT string = "2006-01-02T15:04:05"

	CA_EXPIRE time.Duration = 3153600000 * time.Second // 100年

	UNKNOWN_CERT_TYPE CertType = "UNKNOWN"
	CA                CertType = "CA"
	SERVER            CertType = "SERVER"
	CLIENT            CertType = "CLIENT"
)

// CA証明書を発行します
func CreateCACert(req *CreateCACertRequest) (*CertData, error) {

	created := time.Now()
	expire := created.Add(CA_EXPIRE)
	usage := x509.KeyUsageDigitalSignature |
		x509.KeyUsageCertSign |
		x509.KeyUsageCRLSign

	tpl := &x509.Certificate{
		SerialNumber:          big.NewInt(int64(req.Serial)),
		Subject:               req.Subject,
		NotAfter:              expire,
		NotBefore:             created,
		KeyUsage:              usage,
		IsCA:                  true,
		BasicConstraintsValid: true,
	}

	priv := req.PrivateKey.Key
	pem_data, err := createCertificate(tpl, tpl, priv.Public(), priv)

	if err != nil {
		return nil, err
	}

	data := CertData{
		CAID:           req.CAID,
		Serial:         req.Serial,
		CommonName:     req.Subject.CommonName,
		PrivateKey:     req.PrivateKey,
		Type:           CA,
		PemData:        pem_data,
		Created:        created.Format(DT_FORMAT),
		ExpirationDate: expire.Format(DT_FORMAT),
	}

	return &data, nil
}

// x509.CreateCertificate関数をラッピングし、PEM形式の証明書データを出力します
func createCertificate(template *x509.Certificate, parent *x509.Certificate,
	pub crypto.PublicKey, priv crypto.Signer) (string, error) {

	cert, err := x509.CreateCertificate(
		rand.Reader, template, parent, pub, priv)

	if err != nil {
		return "", err
	}

	block := &pem.Block{
		Type:  "CERTIFICATE",
		Bytes: cert,
	}

	data := pem.EncodeToMemory(block)

	if data != nil {
		return string(data), nil

	} else {
		e := errors.New("PEM形式のデータ変換に失敗しました")
		return "", e
	}
}
```

試しにCA証明書の作成コードを見てみましょう。
引数の `CreateCACertRequest` 構造体はREST APIで投げられたパラメータ等を格納するためのものです。

その後必要な情報を `x509.Certificate` 構造体に詰め込んでいきます。
このあたりはopensslのconfigファイルが対応しているかと思います。

必要な情報を詰め込んだら証明書を作成する関数に値を渡します。
CA証明書ですので、ここでは自己署名を行っています。

## ３．データの格納
２．で作成した証明書のデータ（`CertData` 構造体のポインタ）をDBに格納します。
このとき、秘密鍵はそのまま格納すると危険なので、暗号化してから格納する手順を用意しています。

暗号化の仕組みとして、証明書を用いた公開鍵暗号にしようかと思いましたが、共通鍵暗号（AES暗号のGCMモードを採用）の鍵をパスワードとして保管する仕組みでもそれなりにセキュリティは担保できると考え、AES暗号方式にしました。
AWS KMSのような仕組みは個人で用いるには過剰ですし。

個人で使うシステムはやはり利便性は損ないたくありませんからね。

## ４．証明書データの取り出し
DBで保管しているPEM形式の証明書をAPI経由でレスポンスとして返します。
こうすることで、更新を自動化することも可能ですし、証明書発行サーバを別コンテナとして分割することもできます。

# 苦労話など
今回もハマった点でも記事にすればいいやと思っていたのですが、驚くほどスムーズにコードが書けてしまったのでほとんど話すことがないです😅
強いて言うならコードの設計に一番時間をかけました。


少しだけハマった点はありますので、記載できたらなと思います。

## gorpで複数データをInsert・Update・Deleteするときのポイント

```go:db/repository.go
func (r *Repository) DestroyCA(id string, tca models.TranCAInfo) error {
	var result []models.TranCertificate
	tx, err := r.Begin()

	// （途中省略）

	// 1. 複数DeleteのためのinterfaceなSliceを準備する
	// （複数Insert、Updateでも同じ）
	delete_items := make([]interface{}, len(result))

	for i, v := range result {
		// 2. Deleteはポインタでないといけないが、直接ポインタで渡すと
		// 全て同じアドレス値を取り大変なことになるので、一度値を
		// 別の変数に詰め替える
		item := v

		// 3. interfaceなSliceにポインタを突っ込む
		delete_items[i] = &item
	}

	count, err := tx.Delete(delete_items...)
}
```

今回は複数データを削除するときに使用しました。
gorpでは、Insert・Update・Deleteはinterfaceのポインタを対象のデータとして要求されるのですが、ある型の複数データ（スライス）をinterfaceのポインタのスライスに変換する方法がよくわかっていませんでした。
調べてみたところ、for文の中で一度新しい変数へデータを移し替えて（新しいメモリを取得する）、そのポインタをスライスに格納してやるとうまくいくことがわかりました。
正直どちらでもコストはほとんど変わらないと思いますが。。。
（gorpのコード内でもlistがfor文で回されてDBへリクエストが行われているので、Go内で2度ループが回ってしまうことに変わりはない。）

## ginで普遍的なデータをレスポンスとして返すときの方法

```go:server.go
content_type := "application/x-pem-file; charset=utf-8"
byte_data := []byte(cadata[0].CertData)
c.Data(http.StatusOK, content_type, byte_data)

// こちらでも問題はない
// https://github.com/gin-gonic/gin/issues/468
//c.Header("Content-Type", "application/x-pem-file")
//c.String(http.StatusOK, cadata[0].CertData)
```

色々調べてみましたが、ginのContextを関数レシーバとして取る関数の中に `Data` という関数が存在していました。
この関数を使うとステータスコードとContent-Typeとバイナリデータを一挙にレスポンスできて非常に便利でした。

# 最後に
個人でオレオレ証明書を永続的に複数枚管理したいという需要がどれほどあるかはわかりませんが、そういった人の参考になれば幸いだと考えています。
CRLやOCSPなどの失効周りのことは実装していないですし、RFC5280内の詳細な仕様までは追いきれなかったですが、今回証明書周りのコードを実装したことで証明書に関しての知見がまた広がったかなと思います。

Advent Calendarではゆるい感じで興味を持った技術の発表が続けられたらなと思っています。
また来年も頑張る💪