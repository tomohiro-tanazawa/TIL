公式ページのResource Typesページに、テストの書き方のサンプルが列挙されている。
テストを書いてみて、どのようなコマンドに変換されるのか見てみた。
Linuxコマンドの意味を考えるきっかけにもなるので、Linuxの勉強になると感じた。

#### 基本形
```
describe <リソース名> do
end
```

このようにテスト対象としたいリソースを`describe`の後ろに書く。
ブロック内にはmatcherを記述する。matcherというのは評価値・あるべき状態みたいなもの?？？

#### Gemが指定のバージョンでインストールされているか
```ruby:spec/localhost/sample_spec.rb
describe package('unicorn') do
  it { should be_installed.by('gem').with_version('5.4.1') }
end
```
```terminal:terminal
$ gem list --local | grep -iw unicorn | grep -w "(5.4.1)"
```

#### コマンドの終了ステータスを用いて、コマンド実行の成否をテストする
```ruby
describe command('ls /usr') do
  its(:exit_status) { should eq 0}
end
```
```
$ ls /usr ; echo $?
```

https://shellscript.sunone.me/exit_status.html 

このサイトで終了ステータスという仕組みを初めて知った。`$ ls /usr`というコマンドを実行し、成功(=標準出力が得られた?)なら終了ステータスは0、失敗ならそれ以外の値が返ってくる。


#### ファイルが存在するかテストする
その1: /etc/passwdが存在するか。`$?`が０なら成功、それ以外なら失敗と判定している？(あと、コマンドがこれでいいのか不明)
```ruby
describe file('/etc/passwd') do
  it { should exist}
end
```
```:terminal
$ test -e /etc/passwd ; echo $?
```

その２: .gitignoreにmaster.keyが含まれているか(コマンドがこれでいいのか不明)
```ruby
describe file('/var/www/raisetech-live8-sample-app/.gitignore') do
  its(:content) { should match /config\/master.key/ }
end
```
```
$ cat /var/www/raisetech-live8-sample-app/.gitignore 2> /dev/null || echo -n
```


#### コマンドの標準出力内容をテスト
ファイルの存在確認をテストしているが、それなら上のfileを使ったほうが、コードの趣旨がはっきりすると思った。
もっといろんな用途に使えそう。
```ruby
describe command('ls -al /var/www/raisetech-live8-sample-app/config')do
  its(:stdout) { should match /credentials.yml.enc/}
end
```
```
$ ls -al /var/www/raisetech-live8-sample-app/config | grep credentials.yml.enc
```
