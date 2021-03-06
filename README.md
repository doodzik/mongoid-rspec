# mongoid-rspec

[![Build Status][travis_badge]][travis]
[![Gem Version][rubygems_badge]][rubygems]
[![Code Climate][codeclimate_badge]][codeclimate]

mongoid-rspec provides a collection of RSpec-compatible matchers that help to test mongoid documents.

## Installation

### With Mongoid 4.x

Use mongoid-rspec [2.0.0][mongo4]

    gem 'mongoid-rspec', '~> 2.0.0'

### With Mongoid 3.x

Use mongoid-rspec [1.13.0][mongo3].

    gem 'mongoid-rspec', '~> 1.13.0'

### With Mongoid 2.x

Use mongoid-rspec [1.4.5][mongo2]

    gem 'mongoid-rspec', '1.4.5'

### Configuring

Drop in existing or dedicated support file in spec/support.
i.e: `spec/support/mongoid.rb`

```ruby
RSpec.configure do |config|
  config.include Mongoid::Matchers, type: :model
end
```

If you aren't using rails then you don't have to specify the type.
If you want to know why visit [the rspec documentation](https://relishapp.com/rspec/rspec-rails/docs/directory-structure).

## Matchers

### Association Matchers

```ruby
describe User do
  it { should have_many(:articles).with_foreign_key(:author_id).ordered_by(:title) }

  it { should have_one(:record) }
  #can verify autobuild is set to true
  it { should have_one(:record).with_autobuild }

  it { should have_many :comments }

  #can also specify with_dependent to test if :dependent => :destroy/:destroy_all/:delete is set
  it { should have_many(:comments).with_dependent(:destroy) }
  #can verify autosave is set to true
  it { should have_many(:comments).with_autosave }

  it { should embed_one :profile }

  it { should have_and_belong_to_many(:children) }
  it { should have_and_belong_to_many(:children).of_type(User) }
end

describe Profile do
  it { should be_embedded_in(:user).as_inverse_of(:profile) }
end

describe Article do
  it { should belong_to(:author).of_type(User).as_inverse_of(:articles) }
  it { should belong_to(:author).of_type(User).as_inverse_of(:articles).with_index }
  it { should embed_many(:comments) }
end

describe Comment do
  it { should be_embedded_in(:article).as_inverse_of(:comments) }
  it { should belong_to(:user).as_inverse_of(:comments) }
end

describe Record do
  it { should belong_to(:user).as_inverse_of(:record) }
end

describe Site do
  it { should have_many(:users).as_inverse_of(:site).ordered_by(:email.asc) }
end
```

### Mass Assignment Matcher

```ruby
describe User do
  it { should allow_mass_assignment_of(:login) }
  it { should allow_mass_assignment_of(:email) }
  it { should allow_mass_assignment_of(:age) }
  it { should allow_mass_assignment_of(:password) }
  it { should allow_mass_assignment_of(:password) }
  it { should allow_mass_assignment_of(:role).as(:admin) }

  it { should_not allow_mass_assignment_of(:role) }
end
```

### Validation Matchers

```ruby
describe Site do
  it { should validate_presence_of(:name) }
  it { should validate_uniqueness_of(:name) }
end

describe User do
  it { should validate_presence_of(:login) }
  it { should validate_uniqueness_of(:login).scoped_to(:site) }
  it { should validate_uniqueness_of(:email).case_insensitive.with_message("is already taken") }
  it { should validate_format_of(:login).to_allow("valid_login").not_to_allow("invalid login") }
  it { should validate_associated(:profile) }
  it { should validate_exclusion_of(:login).to_not_allow("super", "index", "edit") }
  it { should validate_inclusion_of(:role).to_allow("admin", "member") }
  it { should validate_confirmation_of(:email) }
  it { should validate_presence_of(:age).on(:create, :update) }
  it { should validate_numericality_of(:age).on(:create, :update) }
  it { should validate_inclusion_of(:age).to_allow(23..42).on([:create, :update]) }
  it { should validate_presence_of(:password).on(:create) }
  it { should validate_presence_of(:provider_uid).on(:create) }
  it { should validate_inclusion_of(:locale).to_allow([:en, :ru]) }
end

describe Article do
  it { should validate_length_of(:title).within(8..16) }
end

describe Profile do
  it { should validate_numericality_of(:age).greater_than(0) }
end

describe MovieArticle do
  it { should validate_numericality_of(:rating).to_allow(:greater_than => 0).less_than_or_equal_to(5) }
  it { should validate_numericality_of(:classification).to_allow(:even => true, :only_integer => true, :nil => false) }
end

describe Person do
   # in order to be able to use the custom_validate matcher, the custom validator class (in this case SsnValidator)
   # should redefine the kind method to return :custom, i.e. "def self.kind() :custom end"
  it { should custom_validate(:ssn).with_validator(SsnValidator) }
end
```

### Accepts Nested Attributes Matcher

```ruby
describe User do
  it { should accept_nested_attributes_for(:articles) }
  it { should accept_nested_attributes_for(:comments) }
end

describe Article do
  it { should accept_nested_attributes_for(:permalink) }
end
```

### Index Matcher

```ruby
describe Article do
  it { should have_index_for(published: 1) }
  it { should have_index_for(title: 1).with_options(unique: true, background: true) }
end

describe Profile do
  it { should have_index_for(first_name: 1, last_name: 1) }
end
```

### Others

```ruby
describe User do
  it { should have_fields(:email, :login) }
  it { should have_field(:s).with_alias(:status) }
  it { should have_fields(:birthdate, :registered_at).of_type(DateTime) }

  # if you're declaring 'include Mongoid::Timestamps'
  # or any of 'include Mongoid::Timestamps::Created' and 'Mongoid::Timestamps::Updated'
  it { should be_timestamped_document }
  it { should be_timestamped_document.with(:created) }
  it { should_not be_timestamped_document.with(:updated) }

  it { should be_versioned_document } # if you're declaring `include Mongoid::Versioning`
  it { should be_paranoid_document } # if you're declaring `include Mongoid::Paranoia`
  it { should be_multiparameted_document } # if you're declaring `include Mongoid::MultiParameterAttributes`
end

describe Log do
  it { should be_stored_in :logs }
end

describe Article do
  it { should have_field(:published).of_type(Boolean).with_default_value_of(false) }
  it { should have_field(:allow_comments).of_type(Boolean).with_default_value_of(true) }
  it { should_not have_field(:allow_comments).of_type(Boolean).with_default_value_of(false) }
  it { should_not have_field(:number_of_comments).of_type(Integer).with_default_value_of(1) }
end
```

## Known issues

accept_nested_attributes_for matcher must test options [issue 91](https://github.com/mongoid-rspec/mongoid-rspec/issues/91).

## Acknowledgement

Thanks to [Durran Jordan][durran] for providing the changes necessary to make
this compatible with mongoid 2.0.0.rc, and for other [contributors](https://github.com/mongoid-rspec/mongoid-rspec/contributors)
to this project.

[durran]: https://github.com/durran
[mongo2]: http://rubygems.org/gems/mongoid-rspec/versions/1.4.5
[mongo3]: http://rubygems.org/gems/mongoid-rspec/versions/1.13.0
[mongo4]: http://rubygems.org/gems/mongoid-rspec/versions/2.0.0

[travis_badge]: http://img.shields.io/travis/mongoid-rspec/mongoid-rspec.svg?style=flat
[travis]: https://travis-ci.org/mongoid-rspec/mongoid-rspec

[rubygems_badge]: http://img.shields.io/gem/v/mongoid-rspec.svg?style=flat
[rubygems]: http://rubygems.org/gems/mongoid-rspec

[codeclimate_badge]: http://img.shields.io/codeclimate/github/mongoid-rspec/mongoid-rspec.svg?style=flat
[codeclimate]: https://codeclimate.com/github/mongoid-rspec/mongoid-rspec
