+++
title = "railsでイミュータブルデータモデルを試してみる"
date = 2021-12-25
[taxonomies]
tags = ["Ruby on Rails"]
[extra]
toc = true
+++

## イミュータブルデータモデルとは
データモデルにおいて、モデルに対してUpdateを行わない手法。[kawasimaさんのscrapbox](https://scrapbox.io/kawasima/%E3%82%A4%E3%83%9F%E3%83%A5%E3%83%BC%E3%82%BF%E3%83%96%E3%83%AB%E3%83%87%E3%83%BC%E3%82%BF%E3%83%A2%E3%83%87%E3%83%AB)が詳しい

## この記事のゴール
railsでDBに対してupdateを行わない方法を見つける


## 環境
* Ruby: 3.1.0
* Rails: 7.0.2.2
* DB: PostgresSQL 14.2

## 題材
なんらかの記事(`post`)とその発行(`post_publish`)、発行の取消し(`post_publish_delete`)について扱う。

![models](/content/immutable-data-model-in-rails/models.svg)

<details> 
<summary>PlantUML記法</summary>

```
@startuml models

Post ||..|{ PostPublish
PostPublish ||..o|PostPublishDelete

@enduml
```
</details>

## 実装
### migration
```ruby
# postのカラムは簡単にするためtitleとauthorのみ
class CreatePosts < ActiveRecord::Migration[7.0]
  def change
    create_table :posts do |t|
      t.string :title, null: false
      t.string :author, null: false

      t.timestamps
    end
  end
end

class CreatePostPublishes < ActiveRecord::Migration[7.0]
  def change
    create_table :post_publishes do |t|
      t.references :post, null: false, foreign_key: { on_delete: :cascade }
      t.date   :published_at, null: false

      t.timestamps
    end

    # 今回はidではなく時刻でイベントの順序を判断している
    add_index :post_publishes, %i[created_at post_id], unique: true
  end
end

class CreatePostPublishDeletes < ActiveRecord::Migration[7.0]
  def change
    create_table :post_publish_deletes do |t|
      t.references :post_publish, null: false, foreign_key: { on_delete: :cascade }

      t.timestamps
    end

    add_index :post_publish_deletes, %i[created_at post_publish_id], unique: true
  end
end

```

### models
#### 最新のレコードがほしい
post - post_publishは1対nの関係を持つが、大抵のケースではpostに紐づく最新のpost_publishのみが必要になるだろう。
この場合、`.with_latest_publish`のようにpost_publishを自己結合させることで最新のpost_publishが結びついたpostを取得することができる(post_publishが結びつかないpostはそのまま取得できる)

```ruby
# app/models/post.rb
class Post < ApplicationRecord

  has_many :post_publishes

  scope :with_latest_publish, lambda {
    joins(<<~SQL)
        LEFT OUTER JOIN post_publishes AS new_post_publishes
          ON post_publishes.post_id = new_post_publishes.post_id
          AND post_publishes.created_at < new_post_publishes.created_at
    SQL
      .where('new_post_publishes.post_id': nil)
      .eager_load(:post_publishes)
  }
end
```

#### publishイベント発生
publishイベントが発生したときは、postインスタンスからpublishインスタンスを生成する。
has_manyに記載したassociation名に`.create`メソッドが使える。今回だと`post_publishes.create`と書ける。このときassociation先のデータが0件でも問題ない。post_publishの方でvalidationを設定しておけばvalidationもしてくれる。

```ruby
# app/models/post.rb
class Post < ApplicationRecord

  has_many :post_publishes

  def publish_at!(at = Date.today, created_at = Time.zone.now)
    post_publishes.create!(published_at: at, created_at: created_at)
  end
end

# app/models/post_publish.rb
class PostPublish < ApplicationRecord
  belongs_to :post

  validate :already_published

  private

  def already_published
    published = self.class.order(created_at: :DESC).find_by(post_id: post_id)

    return if published.blank?
    return if published.post_publish_delete.present?

    errors.add(:already_published, "すでにpublish済みです: #{post_id}")
  end
end
```

#### publishの削除イベント発生
publishを削除するときは、publish_deleteイベントが発生したことにする。直接post_publishをdeleteしてもよいが、イミュータブルデータモデルを採用するときはイベントをログとして残したいはずなので、削除イベントの発生として表現する。
post_publishインスタンスからpost_publish_deleteイベントを生成する(`PostPublish#withdraw`)

ここでpublishに対して削除イベントが1回しか発生しない、とする場合 `has_one` を使うことになるのだが、`post_publish_delete.create` のようにすると `undefined method 'create' for nil:NilClass` が発生する。
代わりに`create_post_publish_delete`と書ける。ただし、すでにpost_publish_deleteが存在する場合は、新規レコードを作成して古いpost_publish_deleteを関連から外すような動きをする。これはpost_publish_deleteモデルでvalidationエラーが発生していても発生する。そのため呼び出し側(`PostPublish#withdraw`)で塞いでおく必要がある。

```ruby
# app/models/post_publish.rb
class PostPublish < ApplicationRecord
  has_one :post_publish_delete

  def withdraw
    raise "すでに削除済みです: (post_publish_id: #{id})" if post_publish_delete.present?

    create_post_publish_delete(post_publish_id: id)
  end
end

# app/models/post_publish_delete.rb
class PostPublishDelete < ApplicationRecord
  belongs_to :post_publish

  validate :cannot_be_updated

  private

  # post_publish.create_publish_deleteの場合このバリデーションでエラーになっても、DBへのinsertは発生する(結局unique制約で例外になる)
  # このvalidationは直接PostPublishDelete.createされた場合の対策
  def cannot_be_updated
    return unless self.class.find_by(post_publish_id: post_publish_id)

    errors.add(:already_deleted, "すでに削除済みです(post_publish_id: #{post_publish_id})")
  end
end
```

### spec
```ruby
# spec/models/post_spec.rb
require 'rails_helper'

describe Post, type: :model do

  describe '.with_latest_publish' do
    subject { described_class.with_latest_publish }

    let(:post) { FactoryBot.create(:post) }
    let!(:only_publish) { post.publish_at! }

    let(:post2) { FactoryBot.create(:post) }
    let!(:multiple_publishes) do
      post2.publish_at!(Date.yesterday, 30.minutes.ago).withdraw
      post2.publish_at!
    end

    let!(:post3) { FactoryBot.create(:post) }

    it do
      expect(subject).to contain_exactly(post, post2, post3)
      expect(subject.map(&:post_publishes).to_a.flatten).to contain_exactly(only_publish, multiple_publishes)
    end
  end

  describe '#published?' do
    subject { post.published? }

    let(:post) { FactoryBot.create(:post) }

    context '一度もpublishされてない場合' do
      it { is_expected.to be false }
    end

    context '最後のpublishがdeleteされている場合' do
      before do
        post.publish_at!.withdraw
        post.publish_at!.withdraw
      end

      it { is_expected.to be false }
    end

    context '最後のpublishがdeleteされていない場合' do
      before do
        post.publish_at!.withdraw
        post.publish_at!
      end

      it { is_expected.to be true }
    end
  end

  describe '#publish_at!' do
    subject { post.publish_at! }
    let(:post) { FactoryBot.create(:post) }

    context '一度もpublishしていない場合' do
      it do
        expect { subject }.to change(PostPublish, :count).by(1)
        expect(subject).to be_a(PostPublish).and be_persisted
      end
    end

    context '最後のpublishがdeleteされている場合' do
      before do
        post.publish_at!.withdraw
        post.publish_at!.withdraw
      end

      it do
        expect { subject }.to change(PostPublish, :count).by(1)
        expect(subject).to be_a(PostPublish).and be_persisted
      end
    end

    context '最後のpublishがdeleteされていない場合' do
      before do
        post.publish_at!.withdraw
        post.publish_at!
      end

      it do
        expect { subject }.to raise_error(ActiveRecord::RecordInvalid, "Validation failed: Already published すでにpublish済みです: #{post.id}")
                                .and change(PostPublish, :count).by(0)
      end
    end
  end
end

# spec/models/post_publish_spec.rb
require 'rails_helper'

describe PostPublish, type: :model do
  describe '#withdraw' do
    subject { post_publish.withdraw }
    let(:post_publish) do
      post = FactoryBot.create(:post)
      post.publish_at!
    end

    context 'すでに削除済みの場合' do
      before { post_publish.withdraw }

      it do
        expect { subject }.to raise_error(RuntimeError, "すでに削除済みです: (post_publish_id: #{post_publish.id})")
                                .and change(PostPublishDelete, :count).by(0)
      end
    end

    context 'まだ削除済みでない場合' do
      it do
        expect { subject }.to change(PostPublishDelete, :count).by(1)
        expect(subject).to be_a(PostPublishDelete).and be_persisted
      end
    end
  end
end

```

