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
### ⑤登録フォームのviewに追加
registrations/new.html.erb
```
<div class='container'>
  <div class='row'>
    <div class="col-sm-12 col-md-8 col-lg-5 px-5 px-sm-0">
      <h2>Sign up</h2>
      <%= form_with model: @user, url: user_registration_path, id: 'new_user', class: 'new_user', local: true do |f| %>
        <%= render "devise/shared/error_messages", resource: resource %>
        <div class="field">
          <%= f.label :name %><br />
          <%= f.text_field :name, autofocus: true, class: "name" %>
        </div>
        <div class="field">
          <%= f.label :email %><br />
          <%= f.email_field :email, autofocus: true, class: "email" %>
        </div>
        <div class="field">
          <%= f.label :postcode %><br />
          <%= f.text_field :postcode, autofocus: true, class: "postcode" %>
        </div>
        <div class="field">
          <%= f.label :prefecture_code %><br />
          <%= f.collection_select :prefecture_code, JpPrefecture::Prefecture.all,:name,:name, {prompt: '都道府県'},{class: "prefecture_code"} %>
        </div>
        <div class="field">
          <%= f.label :address_city %><br />
          <%= f.text_field :address_city, autofocus: true, class: "address_city" %>
        </div>
        <div class="field">
          <%= f.label :address_street %><br />
          <%= f.text_field :address_street, autofocus: true, class: "address_street" %>
        </div>
        <div class="field">
          <%= f.label :password %>
          <% if @minimum_password_length %>
          <em>(<%= @minimum_password_length %> characters minimum)</em>
          <% end %><br />
          <%= f.password_field :password, autocomplete: "off", class: "password" %>
        </div>
        <div class="field">
          <%= f.label :password_confirmation %><br />
          <%= f.password_field :password_confirmation, autocomplete: "off", class: "password_confirm" %>
        </div>
        <div class="actions">
          <%= f.submit "Sign up" ,class: 'btn btn-sm btn-success' %>
        </div>
      <% end %>
      <%= render "devise/shared/links" %>
    </div>
  </div>
</div>
```
### ⑥jpostalの呼び出し
app/assets/javascripts/users.coffee
```
$ ->
  $("#user_postcode").jpostal({
    postcode : [ "#user_postcode" ],
    address  : {
                  "#user_prefecture_code" : "%3",
                  "#user_address_city"            : "%4",
                  "#user_address_street"          : "%5%6%7"
                }
  })
```
### ⑦user controllerに追記
```
  private
  def user_params
    params.require(:user).permit(:name, :introduction, :profile_image,:postcode, :prefecture_name, :address_city, :address_street)
  end
```