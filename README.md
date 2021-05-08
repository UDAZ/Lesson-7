# Lesson-7 住所の自動入力

### ①jquery.jpostal.jsのインストール
https://github.com/ninton/jquery.jpostal.js/
app/assets/javascriptsにjquery.jpostal.jsを配置する

### ②gem JpPrefectureをインストール
gemfile
```
gem 'jp_prefecture'
```
### ③マイグレーションでuserモデルにカラム追加
```
rails g migration AddColumnsToUsers postcode:integer
```
作成されたマイグレーションファイルに記述
```
  def change
    add_column :users, :postcode, :integer
    add_column :users, :prefecture_code, :integer
    add_column :users, :address_city, :string
    add_column :users, :address_street, :string
  end
```
マイグレーション
```
rails db:migrate
```

### ④userモデルに追記
```
  include JpPrefecture
  jp_prefecture :prefecture_code
  
  def prefecture_name
    JpPrefecture::Prefecture.find(code: prefecture_code).try(:name)
  end
  
  def prefecture_name=(prefecture_name)
    self.prefecture_code = JpPrefecture::Prefecture.find(name: prefecture_name).code
  end
```