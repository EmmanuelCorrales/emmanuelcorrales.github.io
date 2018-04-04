---
layout: post
title:  "Ruby on Rails: Setting up a table with multiple columns with the same model"
date:   2018-04-04 17:06:33 +0800
categories: ruby rails
tags: [ ruby, rails ]
---

Generate a User model.
{% highlight shell %}
rails g model User name
{% endhighlight %}

Generate a TransferRequest Model
{% highlight shell %}
rails g model TransferRequest sender:references receiver:references
{% endhighlight %}

The command will generate the codes below.
{% highlight ruby %}
# app/models/transfer_request.rb
class TransferRequest < ApplicationRecord
  belongs_to :sender
  belongs_to :receiver
  belongs_to :inventory
end
{% endhighlight %}


{% highlight ruby %}
# db/migrations/20180404063005_create_transfer_requests.rb
class CreateTransferRequests < ActiveRecord::Migration[5.1]
  def change
    create_table :transfer_requests do |t|
      t.references :sender, foreign_key: true
      t.references :receiver, foreign_key: true
      t.references :inventory, foreign_key: true

      t.timestamps
    end
  end
end
{% endhighlight %}


Remove the 'foreign_key: true' as paramaters for t.references method because
 Rails will look for the table named 'sender' and 'receiver' if not removed.

{% highlight ruby %}
class CreateTransferRequests < ActiveRecord::Migration[5.1]
  def change
    create_table :transfer_requests do |t|
      t.references :sender
      t.references :receiver
      t.references :inventory, foreign_key: true
      t.integer :status
      t.text :note
      t.timestamp :responded_at

      t.timestamps
    end
  end
end
{% endhighlight %}

Run migration for creating a table for TransferRequest.
{% highlight shell %}
rails db:migrate # Same as rake db:migrate
{% endhighlight %}

This piece of code will be added to db/schema.rb.
{% highlight ruby %}
# db/schema.rb
create_table "transfer_requests", force: :cascade do |t|
  t.bigint "sender_id"
  t.bigint "receiver_id"
  t.bigint "inventory_id"
  t.datetime "responded_at"
  t.datetime "created_at", null: false
  t.datetime "updated_at", null: false
  t.index ["inventory_id"], name: "index_transfer_requests_on_inventory_id"
  t.index ["receiver_id"], name: "index_transfer_requests_on_receiver_id"
  t.index ["sender_id"], name: "index_transfer_requests_on_sender_id"
end
{% endhighlight %}

Set the relationship that TransferRequest belongs to a 'sender' and a 'receiver'
 so that Rails will look for columns 'sender_id' and 'receiver_id' instead
 of tables that are named 'sender' and 'receiver'. Pass 'User' as the parameter
 value for class_name so that Rails will know that 'sender' and 'receiver' are
 instances of 'User'.
{% highlight ruby %}
class TransferRequest < ApplicationRecord
  belongs_to :sender, class_name: 'User'
  belongs_to :receiver, class_name: 'User'
  belongs_to :inventory
end
{% endhighlight %}

Set the relationship that the model User has many senders and receivers. Pass
 'TransferRequest' as the parameter value for class_name so that Rails will know
 that 'sender' and 'receiver' are instances of 'TransferRequest'. Pass sender_id
 and receiver_id as parameter value for foreign_key so that Rails can identify
 which column of the TransferRequest table should store the corresponding
 foreign keys.

{% highlight ruby %}
class User < ApplicationRecord
  has_many :sender_transfer_request, class_name: 'TransferRequest',
    foreign_key: 'sender_id'
  has_many :receiver_transfer_request, class_name: 'TransferRequest',
    foreign_key: 'receiver_id'

  validates :email, presence: true, uniqueness: true
  validates :name, presence: true
end
{% endhighlight %}
